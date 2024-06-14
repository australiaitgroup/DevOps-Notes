# Linux and Bash

## 目录

- [Introduce to Linux](#introduce-to-linux)
  - [Overview](#overview)
  - [Quickstart](#quickstart)
- [Bash](#bash)
  - [Basic Commands](#basic-commands)
  - [Common Tools](#common-tools)
  - [Bash in Day-to-Day DevOps](#bash-in-day-to-day-devops)
- [Regex](#regex)
  - [Regex Basics](#regex-basics)
  - [Regex Practice](#regex-practice)

## Introduce to Linux

#### Overview

**Why Linux?**

- **Free:** Install Linux on multiple computers without licensing fees.
- **Highly Secure:** Inherently designed to be secure and less vulnerable to attacks.
- **Flexible:** Allows control over every aspect of the OS, including source code modification.

**Why Linux for DevOps?**

- **Tools and Platforms:** Many open-source projects run on Linux (e.g., Git, Docker, Ansible, Kubernetes).
- **Versatility:** Likely to work with a mix of operating systems, with Linux being a significant part.

**What is Linux?**

- An open-source operating system.
- One of the most popular platforms globally (e.g., Android).

**History of Linux**

- Originated from UNIX, created by Bell Labs developers.
- Linux was developed by Linus Torvalds at the University of Helsinki.

**Pros and Cons**

**Pros:**

- Free
- Portable to any hardware platform
- Continuous operation without rebooting
- Secure, versatile, and scalable
- Short debug time

**Cons:**

- Many different distributions
- Not very user-friendly for beginners
- Trustworthiness of open-source projects

#### Quickstart

**For Linux (OS X, Ubuntu, etc.) Users:**

- Open Terminal on your machine.
- (Optional) Install `zsh` for an enhanced shell experience.
  - [How to configure your macOS Terminal with zsh like a Pro](https://www.freecodecamp.org/news/how-to-configure-your-macos-terminal-with-zsh-like-a-pro-c0ab3f3c1156/)

**For Windows Users:**

- Install Windows Subsystem for Linux (WSL).
  - [WSL Installation Guide](https://docs.microsoft.com/en-us/windows/wsl/install)
- (Optional) Install `zsh` for an enhanced shell experience.
  - [WSL with Oh My Zsh and ConEmu](https://blog.joaograssi.com/windows-subsystem-for-linux-with-oh-my-zsh-conemu/)

##### Directories

- Check the current directory:

```bash
pwd
```

- List the contents of the current directory, including hidden files:

```bash
ls -la
```

- Change to another directory:

```bash
cd <directory_name>
```

##### Files

- Create a new file with content:

```bash
echo "hello world" > /tmp/file
```

- Display the file content:

```bash
cat /tmp/file
```

- Edit the file:

```bash
vim /tmp/file
# To exit: :q
# To save and exit: :wq
```

##### Getting Help

- Read the manual for a command:

```bash
man <command>
```

- Read information for a command:

```bash
info <command>
```

### Bash

#### Basic Commands

Refer to this [Article on Basic Bash Commands](https://github.com/JiangRenDevOps/DevOpsLectureNotesV6/blob/master/WK5_Linux_Bash_Basics/bash_basic_commands.md).

#### Common Tools

- **grep:** [Grep Command](https://www.geeksforgeeks.org/grep-command-in-unixlinux/?ref=lbp)
- **sed:** [Sed Command](https://www.geeksforgeeks.org/sed-command-in-linux-unix-with-examples/?ref=lbp)
- **awk:** [Awk Command](https://www.geeksforgeeks.org/awk-command-unixlinux-examples/?ref=lbp)
- **sort:** [Sort Command](https://www.geeksforgeeks.org/sort-command-linuxunix-examples/?ref=lbp)
- **diff:** [Diff Command](https://www.geeksforgeeks.org/diff-command-linux-examples/?ref=lbp)
- **uniq:** [Uniq Command](https://www.geeksforgeeks.org/uniq-command-in-linux-with-examples/?ref=lbp)

#### Bash in Day-to-Day DevOps

- **Scenario 1:** Securely connect to a remote Linux machine and check the logs.

  - [Launching an Instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/launching-instance.html)
  - [Accessing Instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html)
  - [Access to Logs](https://docs.aws.amazon.com/managedservices/latest/userguide/access-to-logs-ec2.html)

- **Scenario 2:** Download/Upload files from/to a remote Linux server.

  - [SCP Access](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html#AccessingInstancesLinuxSCP)

- **Scenario 3:** Check network connectivity on a Linux machine.

  - [Network Connectivity](https://geekflare.com/linux-test-network-connectivity/)

- **Scenario 4:** Set up a daily job on a Linux machine.
  - [Set Up Cron Job](https://phoenixnap.com/kb/set-up-cron-job-linux)

### Regex

#### Regex Basics

Regular Expressions (regex or regexp) are powerful tools to identify specific patterns in text, useful for validation, web scraping, and syntax validation. Regex is widely used across multiple programming languages.

Refer to this [Article on Regex Basics](https://github.com/JiangRenDevOps/DevOpsLectureNotesV6/blob/master/WK5_Linux_Bash_Basics/regex_basics.md).

#### Regex Practice

Practice regex using the following resources:

- [Regex Practice](https://github.com/JiangRenDevOps/DevOpsLectureNotesV6/blob/master/WK5_Linux_Bash_Basics/regex_practice.md)
- [Regexr](https://regexr.com/)
