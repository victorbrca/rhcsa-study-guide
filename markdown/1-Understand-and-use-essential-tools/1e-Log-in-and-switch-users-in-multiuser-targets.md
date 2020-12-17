1.e Log in and switch users in multiuser targets
===

## Understanding Sell Types

**Login Shells**

A login shell (also an interactive shell) is the first process that executes under your user ID when you log in for an interactive session.

Bash runs the following scripts on login:

* /etc/profile
* The first found of ~/.bash_profile, ~/.bash_login, ~/.profile

**Interactive Shells**

An interactive shell reads commands from user input on a tty. Among other things, such a shell reads startup files on activation, displays a prompt, and enables job control by default. The user can interact with the shell. On entering an interactive terminal, Bash also executes:

* /etc/bash.bashrc
* ~/.bashrc

**Non-interactive shells**

Non-interactive shells do not usually execute startup files, however different shells act differently. Bash always reads `~/.bashrc` when its invoked by rshd or sshd, even if its not interactive (but not if its called as sh). Zsh always reads `~/.zshenv`.

Also note that aliases are not expanded when the shell is not interactive, unless the expand_aliases shell option is set using shopt:

## Changing Users

**Commands:**
- su (1) - run a command with substitute user and group ID
- sudo (8) - execute a command as another user

_**Sudo and su commands and shell type**_

| Command         | Interactive | Login |
| --------------- | ----------- | ----- |
| `su`            | Y           | N     |
| `su -`, `su -l` | Y           | Y     |
| `sudo -i`       | Y           | Y     |

**su vs sudo**
+ sudo - asks for user password
+ su - asks for user password

**Files to know**
+ `~/.bash_history`
+ `~/.bash_logout`
+ `~/.bashrc`
+ `~/.bash_profile`

---
[⬅️ Back](1-Understand-and-use-essential-tools.md)
