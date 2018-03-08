# Debian Linux Utilities

## Table of Contents

1. [Add a new user and provide SSH access to him](#add-new-ssh-user)
2. [Add/Remove an user to 'sudo'(ROOT) user group](#addremove-an-user-to-sudo-root-user-group)
3. [Remove an user from the system completely](#remove-an-user-from-the-system-completely)
4. [Find out useful info of your machine](#find-out-useful-info-of-your-machine)
5. [Add nice features to Vim](#add-nice-features-to-vim)
6. [Enable SHELL colors](#enable-shell-colors)
7. [Change the Hostname name](#change-the-hostname-name)
8. [How to maintain SSH session open](#how-to-maintain-ssh-session-open)
9. [How to download a file and save it on a specific path](#how-to-download-a-file-and-save-it-on-a-specific-path)
10. [How to remove a file that starting with '-' or '--'](#how-to-find-specific-files-in-a-path-and-remove-them)
11. [How to find one or more files](#how-to-find-one-or-more-files)
12. [How to upload a file or a directory on a SSH Server](#how-to-upload-a-file-or-a-directory-on-a-ssh-server)
13. [How to find a service and kill it](#how-to-find-a-service-and-kill-it)
14. [How to easily share a terminal on a machine with others](#how-to-easily-share-a-terminal-on-a-machine-with-others)
15. [How to reconfigure keyboard](#how-to-reconfigure-keyboard)
16. [How to TAR command (c = create, z = gzip, f= file, x = extract, v = verbose)](#how-to-tar-command-c--create-z--gzip-f-file-x--extract-v--verbose)
17. [How to kill more processes](#how-to-kill-more-processes)
18. [How to research a word on several files](#how-to-research-a-word-on-several-files)
19. [How to resize images from terminal](#how-to-resize-images-from-terminal)
20. [How to watch the behaviour of a directory](#how-to-watch-the-behaviour-of-a-directory)
21. [How to send emails from the machine (Internet Site)](#how-to-send-emails-from-the-machine-internet-site)

### Add new SSH user 

This example shows how to add the user "username" and enable him to SSH access.

```bash
sudo adduser --home /home/username username
sudo chown -R username:username /home/username/
sudo chmod 0700 /home/username/.ssh
sudo chmod 0600 /home/username/.ssh/authorized_keys
sudo systemctl reload ssh.service
```
(paste the '```id_rsa.pub```' of the new user into his '```authorized_keys```' file)

### Add/Remove an user to 'sudo' (ROOT) user group

```bash
sudo adduser username sudo
```

```bash
sudo deluser username sudo
```

### Remove an user from the system completely

```bash
sudo deluser --remove-home username
```

### Find out useful info of your machine

```bash
sudo lsb_release -a
```

### Add nice features to Vim

```bash
sudo vim /etc/vim/vimrc
```

Add before the end of the file:

```bash
" To enable Vim colors
syntax on

" Vim jump to the last position when reopening a file
if has("autocmd")
  au BufReadPost * if line("'\"") > 1 && line("'\"") <= line("$") | exe "normal! g'\"" | endif
endif

set showcmd            " Show (partial) command in status line.
set showmatch          " Show matching brackets.
set ignorecase         " Do case insensitive matching
set smartcase          " Do smart case matching
set incsearch          " Incremental search
set mouse=a            " Enable mouse usage (all modes)
set tabstop=3
set shiftwidth=3
set expandtab

" To solve copy/paste problem
set viminfo='100,<1000,s100,h

let &t_SI .= "\<Esc>[?2004h"
let &t_EI .= "\<Esc>[?2004l"

inoremap <special> <expr> <Esc>[200~ XTermPasteBegin()

function! XTermPasteBegin()
  set pastetoggle=<Esc>[201~
  set paste
  return ""
endfunction
```

### Enable SHELL colors

```bash
sudo vim /root/.bashrc
```

Remove all the content and add:

```bash
# ~/.bashrc: executed by bash(1) for non-login shells.

# Note: PS1 and umask are already set in /etc/profile. You should not
# need this unless you want different defaults for root.
# PS1='${debian_chroot:+($debian_chroot)}\h:\w\$ '
# umask 022

PS1='${debian_chroot:+($debian_chroot)}\[\033[01;31m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '

# You may uncomment the following lines if you want `ls' to be colorized:
export LS_OPTIONS='--color=auto -h'
eval "`dircolors`"
alias ls='ls $LS_OPTIONS -h'
alias ll='ls $LS_OPTIONS -lh'
alias l='ls $LS_OPTIONS -lAh'

# Some more alias to avoid mistakes:
alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'

if [ -f /etc/bash_completion.d ] && ! shopt -oq posix; then
   . /etc/bash_completion.d
fi
```

### Change the Hostname name

1. Change opportunely the '```/etc/hosts```' file:
	 ```bash
	 127.0.1.1 <full.qualified.domain.name> <hostname>
   ```

2. Run the command:
   ```bash
   sudo hostnamectl set-hostname <new-hostname>
   ```

### How to maintain SSH session open

```bash
ssh -o ClientAliveInterval 120 user@remote-ssh-server-ip
```

### How to download a file and save it on a specific path

```bash
wget http://URL-to-download.xyz -O <file-path>
```

### How to remove a file that starting with '-' or '--'

```bash
rm -f /path/-filename
```

### How to find specific files in a path and remove them

```bash
cd directory
find . -name ".svn" -exec rm -rf {} \;
```

### How to find one or more files

```bash
find / -name nomefile* -type f -print
```
('```*```' means 'all characters')

### How to upload a file or a directory on a SSH Server
1. Upload a file on the '```tmp```' directory:
   ```bash
   scp nomeFile01.png username@debian.example.org:/tmp
   ```
   
2. Upload a directory on the '```tmp```' directory:
  ```bash
  scp -rp nomeCartella username@debian.example.org:/tmp
  ```

### How to find a service and kill it

1. Find the Service ID that is listening the port 8080:
   ```bash
   netstat -anp | grep 8080
   ```
   
   result:
   
   ```bash
   tcp6       0      0 127.0.0.1:8005          :::*                    LISTEN      23502/java
   ```
   
2. Find the service related to that Service ID: 
   ```bash
   ps aux | grep 23502
   ```

   result:
  
   ```bash
   tomcat7  23502  0.8  0.7 1060792 3656 ?        Sl   15:09   0:33 /usr/lib/jvm/default-java/bin/java -Djava.util.logging.config.file=/var/lib/tomcat7/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djava.awt.headless=true -Xmx128m -XX:+UseConcMarkSweepGC -Djava.endorsed.dirs=/usr/share/tomcat7/endorsed -classpath /usr/share/tomcat7/bin/bootstrap.jar:/usr/share/tomcat7/bin/tomcat-juli.jar -Dcatalina.base=/var/lib/tomcat7 -Dcatalina.home=/usr/share/tomcat7 -Djava.io.tmpdir=/tmp/tomcat7-tomcat7-tmp org.apache.catalina.startup.Bootstrap start
   ```

3. Kill it:
   ```bash
   kill -9 23502
   ```

### How to easily share a terminal on a machine with others

1. Install '```byobu```':
   ```bash
   apt install byobu
   ```

2. Run '```byobu```' as sudo user from each person.

### How to reconfigure keyboard

```bash
sudo dpkg-reconfigure keyboard-configuration
```

### How to TAR command (c = create, z = gzip, f= file, x = extract, v = verbose)

1. Create 'archivio.tar.gz' of a directory:
   ```bash
	 tar czf nomeArchivio.tar.gz directory/
   ```
   
2. Create 'archivio.tar.gz' from a group of files:
   ```bash
	 tar czf nomeArchivio.tar.gz pathFile1 pathFile2 pathFile3
   ```
   
3. Estract archivio.tar.gz:
   ```bash
	 tar xzvf archivio.tar.gz
   ```

### How to kill more processes

1. Find processes to kill:
   ```bash
   ps -ef | pgrep -f python
   ```

2. Kill them:
   ```bash
   ps -ef | pkill -f python
   ```

### How to research a word on several files

```bash
grep -rnw '/path/to/somewhere/' -e "pattern"
```

### How to resize images from terminal

1. Install '```imagemagick```':
   ```bash
   sudo apt-get install imagemagick
   ```

2. Resize image without lose quality (by respecting aspect-ratio):
   ```bash
   convert logo.png -resize 160x120 logo.png
   ```

### How to watch the behaviour of a directory
```bash
watch -n 1 "ls -altr /tmp"
```

### How to send emails from the machine (Internet Site)

1. Install needed packages:
   ```bash
   sudo apt install mailutils postfix
   ```
   
2. Try to send a message:
   ```bash
   echo "Body Text" | mail -s "Mail Subject" your.real.email@domain.changeme
   ```
