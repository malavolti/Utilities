# Debian Linux Utilities

## Table of Contents

1. [How to add a new user and provide SSH access to him](#how-to-add-a-new-user-and-provide-ssh-access-to-him)
2. [How to add a new system user and provide SSH access to him](#how-to-add-a-new-system-user-and-provide-ssh-access-to-him)
3. [How to add/remove an user to 'sudo'(ROOT) user group](#how-to-addremove-an-user-to-sudoroot-user-group)
4. [How to remove an user from the system completely](#how-to-remove-an-user-from-the-system-completely)
5. [How to find out useful info of your machine](#how-to-find-out-useful-info-of-your-machine)
6. [How to add nice features to Vim](#how-to-add-nice-features-to-vim)
7. [How to enable SHELL colors](#how-to-enable-shell-colors)
8. [How to change the Hostname name](#how-to-change-the-hostname-name)
9. [How to maintain SSH session open](#how-to-maintain-ssh-session-open)
10. [How to download a file and save it on a specific path](#how-to-download-a-file-and-save-it-on-a-specific-path)
11. [How to remove a file that starting with '-' or '--'](#how-to-find-specific-files-in-a-path-and-remove-them)
12. [How to find one or more files](#how-to-find-one-or-more-files)
13. [How to upload a file or a directory on a SSH Server](#how-to-upload-a-file-or-a-directory-on-a-ssh-server)
14. [How to find a service and kill it](#how-to-find-a-service-and-kill-it)
15. [How to easily share a terminal on a machine with others](#how-to-easily-share-a-terminal-on-a-machine-with-others)
16. [How to reconfigure keyboard](#how-to-reconfigure-keyboard)
17. [How to TAR command (c = create, z = gzip, f= file, x = extract, v = verbose)](#how-to-tar-command-c--create-z--gzip-f-file-x--extract-v--verbose)
18. [How to kill more processes](#how-to-kill-more-processes)
19. [How to research a word on several files](#how-to-research-a-word-on-several-files)
20. [How to resize images from terminal](#how-to-resize-images-from-terminal)
21. [How to watch the behaviour of a directory](#how-to-watch-the-behaviour-of-a-directory)
22. [How to send emails from the machine (Internet Site)](#how-to-send-emails-from-the-machine-internet-site)
23. [How to format and mount a new hard disk](#how-to-format-and-mount-a-new-hard-disk)
24. [How to remove a partition from an hard disk](#how-to-remove-a-partition-from-an-hard-disk)
25. [How to create and use a swapfile](#how-to-create-and-use-a-swapfile)
26. [How to comment/uncomment an entire block of text on Vim](#how-to-commentuncomment-an-entire-block-of-code-on-vim)
27. [How to remove files older than one year](#how-to-remove-files-older-than-one-year)
28. [How to copy directory without losing timestamps](#how-to-copy-directory-without-losing-timestamps)
29. [How to find something with grep by excluding multiple things](#how-to-find-something-with-grep-by-excluding-multiple-things)
30. [How to copy text files without losing data](#how-to-copy-text-files-without-losing-data)
31. [How to format an XML file](#how-to-format-an-xml-file)
32. [How to check certificate on specific port](#how-to-check-certificate-on-specific-port)
33. [How to fix warning: setlocale: LC_ALL: cannot change locale](#how-to-fix-warning-setlocale-lc_all-cannot-change-locale)
34. [How to add new GPG key for APT on Debian 11+](#how-to-add-new-gpg-key-for-apt-on-debian-11)
35. [Howt to customize the prompt of Bash](#how-to-customize-the-prompt-of-bash)

### How to add a new user and provide SSH access to him 

This example shows how to add the user "username" and enable him to SSH access.

```bash
sudo useradd -m /home/username -g username -s /bin/bash
sudo mkdir /home/username/.ssh
sudo chown username:username /home/username/.ssh
sudo chmod 0700 /home/username/.ssh
sudo touch /home/username/.ssh/authorized_keys
sudo chown username:username /home/username/.ssh/authorized_keys
sudo chmod 0600 /home/username/.ssh/authorized_keys
sudo systemctl reload ssh.service
```
(paste the '```id_rsa.pub```' of the new user into the '```authorized_keys```' file)

### How to add a new system user and provide SSH access to him 

This example shows how to add the user "username" and enable him to SSH access.

```bash
sudo useradd -m /home/username -g username -s /bin/bash --system
sudo mkdir /home/username/.ssh
sudo chown username:username /home/username/.ssh
sudo chmod 0700 /home/username/.ssh
sudo touch /home/username/.ssh/authorized_keys
sudo chown username:username /home/username/.ssh/authorized_keys
sudo chmod 0600 /home/username/.ssh/authorized_keys
sudo systemctl reload ssh.service
```
(paste the '```id_rsa.pub```' of the new user into the '```authorized_keys```' file)

### How to add/remove an user to 'sudo'(ROOT) user group

```bash
sudo usermod -a -G sudo username
```

```bash
sudo gpasswd -d username sudo
```

### How to remove an user from the system completely

```bash
sudo apt install perl-modules
sudo deluser --remove-home username
```

### How to find out useful info of your machine

```bash
sudo lsb_release -a
```

### How to add nice features to Vim

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

### How to enable SHELL colors

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

### How to change the Hostname name

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
### How to format and mount a new hard disk

1. Find out the disk with:
   * `fdisk -l`
   
2. Create a new partition on that disk:
   * `fisk /dev/sdb`
     * press '`n`' (to create a new partition)
     * press '`p`' (to select to create a new primary partition)
     * press '`1`' (to select to create only one partition)
     * press '`Enter`' more than one time to keep the default values proposed
     * press '`wq`' + '`Enter`' to write changes on the partitions table.
     
3. Format the new partition into EXT4:
   * `mkfs.ext4 /dev/sdb1`
   
4. Create the directory where the new hard disk will be mounted:
   * `mkdir /data`

5. Modify the `/etc/fstab` to made changes permanent
   * `echo /dev/sdb1 /data ext4  defaults,noatime,nodiratime,relatime 0 0 >> /etc/fstab`

6. Mount the new Hard disk into the destination directory:
   * `mount -a`

### How to remove a partition from an hard disk

1. Unmount the partition
   * `umount /dev/sdb1`

2. Remove the partition
   * `fdisk /dev/sdb`
     * press '`d`' (to delete partition)
     * press '`wq`' + '`Enter`' to write changes on the partitions table.

3. Modify the `/etc/fstab` to made changes permanent by removing his line.

### How to create and use a swapfile

1. Reserve space for the SWAP file (1M * 2048 = 2048 MB = 2 GB):
   * `sudo dd if=/dev/zero of=/swapfile bs=1M count=2048`

2. Create the swapfile:
   * `sudo chmod 600 /swapfile`
   * `sudo mkswap /swapfile`
  
3. Modify the `/etc/fstab` to made changes permanent
   * `sudo echo '/swapfile  none  swap  sw 0 0' >> /etc/fstab`
  
4. Enable SWAP (use '`swapon -s` to check that works)
  `sudo swapon /swapfile`

### How to comment/uncomment an entire block of code on Vim

For commenting a block of text:
1. Put your cursor on the first character you want to comment
2. Press 'Ctrl+V' and go up/down until the last commented line
3. Press 'Shift+I' and then press '#'. This will add a hash to the first line.
4. Press Esc (give it a second) and it will insert a # character on all other selected lines.

For uncommenting a block of text:
1. Put your cursor on the first character you want to uncomment
2. Press 'Ctrl+V' and go down until the last commented line and press 'x', that will delete all the # characters vertically.

### How to remove files older than one year

`find /path/to/files/* -mtime +365 -delete`

### How to copy directory without losing timestamps

`sudo cp -rp /source/path /dest/path`

### How to find something with grep by excluding multiple things

This grep will find the word "beta" in the current directory `.` without considering `name-dir-1,name-dir-2,name-dir-3` and hidden files, XML files or `error_log` file.
The `name-dir-*` is not the path, but the name of the directory:
* path = `/opt/project/exampleDir`
* name-dir = `exampleDir`

`grep -r --exclude-dir={name-dir-1,name-dir-2,name-dir-3} --exclude={.*,*.xml,error_log} "beta" .`

If you need to exclude only one thing:

`grep -r --exclude-dir={name-dir-1,} --exclude={.*,} "beta" .`

### How to copy text files without losing data

1. Create the base64 version of the file: 
   ```bash
   base64 file.ext
   ```

2. Copy the output generated

3. Generate the new file from the Base64 value:
   * `vim encoded_file.ext`
   * Paste the content copied and save it on the file
   * `base64 -d encoded_file.ext > file.ext`

### How to format an XML file

* `xmllint --format ugly.xml --output pretty.xml`

### How to check certificate on specific port

* `openssl s_client -connect www.paypal.com:443`

### How to fix warning: setlocale: LC_ALL: cannot change locale

* `sudo locale -a`   (and discover which locale you have enabled)
* `sudo vim /etc/environment` (and paste into `LANG` and `LC_ALL` the locale you want)
  
  ```bash
  LANG=en_GB.utf8
  LC_ALL=en_GB.utf8
  ```

* Then exit and enter again to server

IF `sudo locale -a` returns only `C`, `C.UTF-8` and `POSIZ` run the following commands:

* `sudo vim /etc/locale.gen` (and uncomment the locales preferred)
* `sudo locale-gen` (to generate locales selected)
* `sudo locale -a` (to check the locales generation)

### How to add new GPG key for APT on Debian 11+

* Download the PUB key, convert into a GPG key and save it on `/usr/share/keyrings` directory
  * `sudo wget -O- https://dl.google.com/linux/linux_signing_key.pub | gpg --dearmor | sudo tee /usr/share/keyrings/google-chrome.gpg`
* Importa il repository che usa la nuova GPG in `/etc/apt/sources.list.d`
  * `echo deb [arch=amd64 signed-by=/usr/share/keyrings/google-chrome.gpg] http://dl.google.com/linux/chrome/deb/ stable main | sudo tee /etc/apt/sources.list.d/google-chrome.list`

### How to customize the prompt of bash

* Enjoy with: https://scriptim.github.io/bash-prompt-generator/

* Bash prompt preferred to put into `~/.bashrc` to permanently set:
  * For the 'root' user:
    
    ```bash
    PS1='\[\e[0;1;91m\]\u\[\e[0;1;91m\]@\[\e[0;1;91m\]\h\[\e[0m\][\[\e[0;38;5;226m\]\W\[\e[0m\]]\[\e[0m\]:\[\e[0m\]'
    ```
    
  * For the other users:
  
    ```bash
    PS1='\[\e[0;1;38;5;112m\]\u\[\e[0;1;38;5;112m\]@\[\e[0;1;38;5;76m\]\h\[\e[0m\][\[\e[0;38;5;226m\]\W\[\e[0m\]]\[\e[0m\]:\[\e[0m\]'
    ```
