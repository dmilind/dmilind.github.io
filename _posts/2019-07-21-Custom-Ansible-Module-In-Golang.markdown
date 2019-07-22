---
title: "Custom Ansible Module In Golang"
layout: post
date: 2019-07-21
image: /assets/images/markdown.jpg
headerImage: false
tag:
- go
- golang
- go ansible module
- custom ansible module
category: blog
author: Milind
description:
---

## Custom Ansible Module In Golang

Ansible is a configuration management tool which can push configuration for the services. Ansible comes with thousands of default modules which can be invoked in the playbook. Sometimes there would be a need to write custom modules in any programming language ( as per ansible says on official documentation ). For writing the module I chose `go` language since I started learning it and I am loving its simplicity.

#### What is this blog for ?

To answer this question, I have to tell you about my problem statement. While working in office, I got a tasks assigned to me which was to refactor two factor authentication (yubikey) on remote host using ansible. I started understanding the process, then I realized that I can make this simple if I wrote a custom ansible module in `go` and invoke that module in ansible. Then I read documentations from ansible about writing custom modules. While surfing on internet sadly I found only one link related to developing custom ansible module in `go`. But that was not wide clear to understand. That time only I decided to pen down my stuff while developing the module with detail explanation so that I can save someones time.
So in nutshell, this blog is describing the development of custom ansible module in `go`.

#### What is going to achieve ?
For this practice I am going to set up two factor authentication using google authenticator. Developed module can be invoked in ansible to set up google authorization.

#### Problem Statement ?

Remote server should not be accessed easily with password. But it should be accessed with a combination of public key as well as authentication code provided by google authenticator app on the phone.

#### Requirements

1.<b> Vagrant :</b>

  This is our sandbox environment which can be destroyed easily and nothing will be harmed. Since vagrant works with virtualbox so, preinstalled virtualbox is expected.

2.<b> Ansible :</b>

  Of course ansible would be required since module is being written for ansible. Installation of ansible is expected on the local machine.

3.<b> Go :</b>

  Custom module is written in `go` so its installation is also expected.

#### Lets start . . .
Instead of writing this blog in much technical depth , I would like to keep this simple so that everyone can understand.

#### Provisioning sandbox environment:
We need to get our sandbox environment up and running. I am using centos 7 vagrant box for this. Limited changes I have made in the Vagrantfile. You can get this <a href='https://github.com/dmilind/golang/blob/master/codingFun/functions/ansible-module/vagrant/Vagrantfile'> <b> Vagrantfile </b> </a> from my github repo.

Now lets spin this machine.
<a href="https://asciinema.org/a/j0TQwXQvjMcXZuwzooNRFjkM8" target="_blank"><img src="https://asciinema.org/a/j0TQwXQvjMcXZuwzooNRFjkM8.svg" /></a>
You can ssh to this machine since public key will be used to authentication.

Lets write basic ansible playbook to configure this server as per our requiorement.

#### Configuring sandbox with minimal configurations:
Ansible directory structure would be like this.
```bash
mdhoke＠ansible[master] ➤ tree
.
├── inventory
├── roles
│   ├── common
│   │   └── tasks
│   │       └── main.yaml
│   └── googleauth
│       ├── library
│       │   ├── googleAuth
│       │   └── googleAuth.go
│       └── tasks
│           └── main.yaml
└── site.yaml

6 directories, 6 files

````
* Role `common` is required to push all common configurations to the test server.
* once all tasks have been executed from this role, then `googleauth` will be called to set up two factor authentication.

You can find these <a href='https://github.com/dmilind/golang/tree/master/codingFun/functions/ansible-module/ansible'><b> Ansible Playbooks </b></a> from my github:


```yaml
# Ansible playbook: Using custom ansible module written in golang
---
- hosts: all
  gather_facts: true
  become: yes
  roles:
    - { role: common, tags: ['common'] }
```
For now consider that we have not created a role for `googleauth`. Our server has been set up with all prerequisites coming from `common` role.

#### Developing an ansible role googleauth

`googleAuth` role should have below directory structure.

```bash
.
├── library
│   ├── googleAuth
│   └── googleAuth.go
└── tasks
    └── main.yaml
```

* `main.yaml` file will have a task to invoke a module `googleAuth` and pass an attribute to the role.
* This module will take this attribute and performs the actions. Module will either enable or disable the two factor authentication. Expected attribute value should be `enable` or `disable`.

#### Developing custom module googleAuth

<b>Key Points:</b>
* Ansible takes binary as a module and executes that one when module is invoked. That being said, `go` binary should be made available under library.
* When tasks is being executed, ansible generates a temporary JSON file which will be made available to this binary. So our binary should takes this file as an argument and then retrieves the attribute. On that basic program can be executed from the binary.
* Ansible takes a return data from binary in the JSON format only. So our code should provide the data in json format.

I tried to explain the code at every step with proper documentation as well. Try to understand it.

```go
/*
Custom ansible module written in golang. This program can be used directly either
by using go tool (go run) or exeuting binary built by go tool (go build)
*/
package main

import (
	"encoding/json"
	"fmt"
	"io"
	"io/ioutil"
	"os"
	"os/exec"
	"strings"
)

// global variable declaration
var (
	returnResponse []byte
	response ModuleResponse

)

// ModuleArgs to collect the state provided in argument file, this file will be generated by ansible when module is invoked.
type ModuleArgs struct {
	State string
}

// ModuleResponse struct to provide json formatted data to ansible, ansbile expects json data in return
type ModuleResponse struct {
	Msg     string `json:"msg"`
	Failed  bool   `json:"failed"`
	Changed bool   `json:"changed"`
}

// ReturnResponse function prints json data , data is collected from struct ModuleResponse
func ReturnResponse(response ModuleResponse) {
	retVal, _ := json.Marshal(response)
	fmt.Println(string(retVal))
}

// FileEdit function to edit the contents of a provided file. This function take 3 arguments "filename", "oldline", "newline" all of string type.
// "filename" is the file which will be edited, "oldline" is the line to be edited, "newline" is the line to be added
func FileEdit(filename, oldLine, newLine string) {
	input, err := ioutil.ReadFile(filename)
	if err != nil {
		fmt.Println("Error while reading a file -->", filename)
		os.Exit(1)
	}
	lines := strings.Split(string(input), "\n")
	for i, line := range lines {
		if strings.Contains(line, oldLine) {
			lines[i] = newLine
		}
	}
	output := strings.Join(lines, "\n")
	err = ioutil.WriteFile(filename, []byte(output), 0644)
	if err != nil {
		fmt.Println("Error while writing a file -->", filename)
		os.Exit(1)
	}
}

// FileCopy function copies a content of source file to destination file. Function accepts 2 arguments src, and dest both of string type.
func FileCopy(src, dest string) {
	srcFile, err := os.Open(src)
	if err != nil {
		fmt.Println("Error while opening a file", src)
		os.Exit(1)
	}
	defer srcFile.Close()

	destFile, err := os.Create(dest)
	if err != nil {
		fmt.Println("Error while creating a file -->", dest)
		os.Exit(1)
	}
	defer destFile.Close()

	_, err = io.Copy(destFile, srcFile)
	if err != nil {
		fmt.Println("Error while copying a file -->", src)
		os.Exit(1)
	}
}

// Executor function to execute system command on the machine.
func Executor(cmd string) {
	output, err := exec.Command(cmd).Output()
	if err != nil {
		fmt.Println("Error while running command", cmd)
		os.Exit(1)
	}
	fmt.Println(string(output)) // change later
}

// EnableGoogleAuth function will enable google authentication.
func EnableGoogleAuth() {
	output, err := exec.Command("/bin/google-authenticator", "-t", "-d", "-f", "-r", "3", "-R", "30", "-W").Output()
	if err != nil {
		fmt.Println("Error while running command --> google-authenticator")
		os.Exit(1)
	}
	google_secrets := string(output)
	// fmt.Println("copying file a file --> /etc/pam.d/sshd")
	FileCopy("/etc/pam.d/sshd", "/home/vagrant/sshd.back")

	// fmt.Println("copying file 2")
	FileCopy("/etc/ssh/sshd_config", "/home/vagrant/sshd_config.back")

	// fmt.Println("editing file 1")
	FileEdit("/etc/pam.d/sshd", "auth       substack     password-auth", "#auth       substack     password-auth")

	// fmt.Println("adding a line in a file --> /etc/pam.d/sshd")
	LineInPam := "auth required pam_google_authenticator.so nullok"
	file, err := os.OpenFile("/etc/pam.d/sshd", os.O_APPEND|os.O_WRONLY, 0644)
	if err != nil {
		fmt.Println("Error while opening a file --> /etc/pam.d/sshd")
		os.Exit(1)
	}

	defer file.Close()
	_, err = file.WriteString(LineInPam)
	if err != nil {
		fmt.Println("Error while writing into a file --> /etc/pam.d/sshd")
		os.Exit(1)
	}

	// fmt.Println("Editing a file --> /etc/ssh/sshd_config")
	FileEdit("/etc/ssh/sshd_config", "#ChallengeResponseAuthentication yes", "ChallengeResponseAuthentication yes")

	// fmt.Println("Editing a file --> /etc/ssh/sshd_config")
	FileEdit("/etc/ssh/sshd_config", "ChallengeResponseAuthentication no", "#ChallengeResponseAuthentication no")

  LineInSSH := "ClientAliveInterval 120\n"
  file, err = os.OpenFile("/etc/ssh/sshd_config", os.O_APPEND|os.O_WRONLY, 0644)
  if err != nil {
    fmt.Println("Error while opening a file --> /etc/ssh/sshd_config")
    os.Exit(1)
  }

  defer file.Close()
  _, err = file.WriteString(LineInSSH)
  if err != nil {
    fmt.Println("Error while writing a file --> /etc/ssh/sshd_config")
    os.Exit(1)
  }

  LineInSSH = "ClientAliveCountMax 2\n"
  file, err = os.OpenFile("/etc/ssh/sshd_config", os.O_APPEND|os.O_WRONLY, 0644)
  if err != nil {
    fmt.Println("Error while opening a file --> /etc/ssh/sshd_config")
    os.Exit(1)
  }

  defer file.Close()
  _, err = file.WriteString(LineInSSH)
  if err != nil {
    fmt.Println("Error while writing a file --> /etc/ssh/sshd_config")
    os.Exit(1)
  }

	// fmt.Println("Adding a line in a file --> /etc/ssh/sshd_config")
	LineInSSH = "AuthenticationMethods publickey,keyboard-interactive\n"
	file, err = os.OpenFile("/etc/ssh/sshd_config", os.O_APPEND|os.O_WRONLY, 0644)
	if err != nil {
		fmt.Println("Error while opening a file --> /etc/ssh/sshd_config")
		os.Exit(1)
	}

	defer file.Close()
	_, err = file.WriteString(LineInSSH)
	if err != nil {
		fmt.Println("Error while writing a file --> /etc/ssh/sshd_config")
		os.Exit(1)
	}

	fmt.Println("Restarting sshd service")
	_, err = exec.Command("systemctl", "restart", "sshd").Output()
	if err != nil {
		fmt.Println("Error while starting sshd daemon")
		os.Exit(1)
	}
	response.Msg = google_secrets
	response.Failed = false
	response.Changed = true
	ReturnResponse(response)
}

// DisableGoogleAuth function disables google authentication also removes all configurations of google authenticator
func DisableGoogleAuth() {
	// copy back up files and restart sshd :) simple
	FileCopy("/home/vagrant/sshd.back", "/etc/pam.d/sshd")
	FileCopy("/home/vagrant/sshd_config.back", "/etc/ssh/sshd_config")

	_, err := exec.Command("systemctl", "restart", "sshd").Output()
	if err != nil {
		fmt.Println("Error while starting sshd daemon")
		os.Exit(1)
	}
	response.Msg = "Google two factor authentication is disabled"
	response.Failed = false
	response.Changed = true
	ReturnResponse(response)
}
func main() {
	// check length of the arguments
	if len(os.Args) != 2 {
		response.Msg = "Missing argument file"
		response.Failed = true
		response.Changed = false
		ReturnResponse(response)
		os.Exit(1)
	}

	argFile := os.Args[1]
	// this is file provide by the ansible at ansible's execution runtime
	// now we need to read this file to get the state which is mentioned in ansible play.
	State, err := ioutil.ReadFile(argFile)
	if err != nil {
		response.Msg = "could not read argument file"
		response.Failed = true
		response.Changed = false
		ReturnResponse(response)
	}
	// state contains byte of data which should be converted into actual string.
	// this data is coming from json file, for that we need to use unmarshal function to
	// get this data into a struct.
	var modArg ModuleArgs
	err = json.Unmarshal(State, &modArg)
	if err != nil {
		response.Msg = "error occured while unmarshling"
		response.Failed = true
		response.Changed = false
		ReturnResponse(response)
	}
	// use switch statement to call appropriate function to enable or disable TFA
	switch modArg.State {
	case "enable":
		// check google authentication status
		// if enabled: --> msg that it is already enabled exit 0
		// calling EnableGoogleAuth function
		EnableGoogleAuth()

	case "disable":
		// if disable: --> msg that it is already disabled exit 0
		// calling DisableGoogleAuth Function
		DisableGoogleAuth()

	default:
		response.Msg = "state attribute takes either enable or disable value"
		response.Failed = true
		response.Changed = false
		ReturnResponse(response)
		os.Exit(1)
  }
}
```
Since ansible expects a binary under the library, let's build a binary for Linux environment as our sandbox is a Linux machine. From library, run below command.
``` go
env GOOS=linux go build googleAuth.go
```
After the successful execution of this command you can find a binary under library directory. We are almost done now.

#### Developing ansible role for googleauth.
Now let's create an ansible role for invoking our developed module.

```yaml
# Playbook responsible to set up google's two factor auth on machine
---
  - name: Adding binary
    copy:
      src: library/googleAuth
      dest: /home/vagrant/

  - name: Configuring Google's Two Factor Authentication
    googleAuth:
      state: enable

  - name: Placing authenticator for users
    shell: |
      cp /root/.google_authenticator /home/"{{ item }}"/
    with_items:
      - vagrant

  - name: Keeping permissions to .google_authenticator
    file:
      path: /home/vagrant/.google_authenticator
      owner: vagrant
      group: vagrant

  - name: Fetch googleauthenticator file locally
    fetch:
      src: /root/.google_authenticator
      dest: /tmp/
      flat: yes
```
As you can see, attribute value provided to `state` is `enable` that tells module to enable two factor authentication.
Now let's run the playbook on this sandbox environment. At this time we don't have any two factor auth setup so we can ssh to this machine using public key only.

<a href="https://asciinema.org/a/1lMl5N9RZ2S8txJqhnFS9NpBm" target="_blank"><img src="https://asciinema.org/a/j0TQwXQvjMcXZuwzooNRFjkM8.svg" /></a>


As you can see our playbook executed successfully , it invoked our module. Let's see do we have two factor authentication or not.

<a href="https://asciinema.org/a/ilzBNsAFvnchcV6HtLQHZTQN6" target="_blank"><img src="https://asciinema.org/a/j0TQwXQvjMcXZuwzooNRFjkM8.svg" /></a>

well it worked 😉

Now question would be, from where I got the verification code to ssh to this machine? As you can see in the playbook,`.google_authenticator` has been fetched and saved under `/tmp/` directory on local machine. This file contents all secret codes. Make sure this file is hidden and not made available to others if you want to restrain access to a machine.

Once you get this file then perform following actions.

1. Download an app called google authenticator on your mobile phone.
2. Open app, touch <b>+</b> from right corner of the screen.
3. Choose `Manual entry` option from the pop up.
4. Open a file `.google_authenticator` and find very first string.
5. Punch that alphanumeric string under `key` section.

Now every 30 sec you will get a new code for accessing this machine.
Hope I tries to explain the things and you can find this useful.
