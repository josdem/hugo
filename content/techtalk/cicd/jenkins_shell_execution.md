+++
title =  "Jenkins Shell Execution"
description = "Jenkins Shell Execution"
date = "2019-02-24T12:14:45-05:00"
tags = ["josdem", "techtalks","programming","technology"]
categories = ["techtalk", "code"]
+++

In this technical post we will cover required configuration we need to setup in our servers in order to execute shell commands or shell scripts using [Jenkins](https://jenkins.io/). You can execute shell in Jenkins as build steps like this.

```bash
echo "Hello World!"
```

*Output*

```bash
[jenkins-sandbox] $ /bin/sh -xe /tmp/jenkins8545009836235168037.sh
+ echo Hello World!
Hello World!
Finished: SUCCESS
```

If you need to execute a shell command with root privileges

```bash
sudo systemctl start jugoterapia-webflux
```

Then you have to add jenkins user to the sudoers group `/etc/sudoers`

```bash
jenkins ALL=(root) NOPASSWD: /bin/systemctl
```

where:

```bash
user host=(root) NOPASSWD: /path/command
```

With previous action our server will not ask for password when we execute that command.

**Remote Execution**

If you need to execute a shell script in another server, then ssh is your best friend.

```bash
ssh josdem@sonar.josdem.io "/home/josdem/jenkins/hello.sh"
```

where:

```bash
ssh user@remote_host "/path/command"
```

*Output*

```bash
jenkins@b612:~$ ssh josdem@sonar.josdem.io "/home/josdem/jenkins/hello.sh"
Hello World!
```

In order to execute previous command you need to create a ssh password less login strategy using [SSH Keygen](https://www.ssh.com/ssh/keygen/)

```bash
ssh-keygen -t rsa
```

*Output*

```bash
Generating public/private rsa key pair.
Enter file in which to save the key (/home/jenkins/.ssh/id_rsa): [Press enter key]
Created directory '/home/jenkins/.ssh'.
Enter passphrase (empty for no passphrase): [Press enter key]
Enter same passphrase again: [Press enter key]
Your identification has been saved in /home/jenkins/.ssh/id_rsa.
Your public key has been saved in /home/jenkins/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:Ts1IUtU48Vb5dVdGY8PD7eWyGcjOBzga6rb9Y7bCZVA jenkins@b612
The key's randomart image is:
+---[RSA 2048]----+
|          .. oo=*|
|         .M = o=X|
|        .. + + +B|
|       oo+o * o +|
|      ..S+o= . = |
|      .+. o o +  |
|     . ..o   .   |
|      o.o +      |
|     ....=oo     |
+----[SHA256]-----+
```

Now you are good to go and copy `/home/jenkins/.ssh/id_rsa.pub` to `remote_server/.ssh/authorized_keys`, Then In Jenkins you can setup that shell command:

```bash
ssh josdem@sonar.josdem.io "/home/josdem/jenkins/hello.sh"
```

*Output*

```bash
[jenkins-sandbox] $ /bin/sh -xe /tmp/jenkins336195165813846731.sh
+ ssh josdem@sonar.josdem.io /home/josdem/jenkins/hello.sh
Hello World!
Finished: SUCCESS
```

[Return to the main article](/techtalk/continuous_integration_delivery)
