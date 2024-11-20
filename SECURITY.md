# SECURITY

# Perform security administration tasks
## Introduction


Security is a must in system administration. As a good Linux sysadmin you have to keep an eye on
a number of things such as special permissions on files, user password aging, open ports and
sockets, limiting the use of system resources, dealing with logged-in users, and privilege escalation
through su and sudo. In this lesson we will be reviewing each one of these topics.

### Checking for Files with the SUID and SGID Set
Aside from the traditional permission set of read, write and execute, files in a Linux system may
also have some special permissions set such as the SUID or the SGID bits.
The SUID bit will allow the file to be executed with the privileges of the file’s owner. 
*The actual use of setting the SUID bit on a file is to allow a non-privileged user to execute a program with the privileges of the file’s owner*


It is numerically represented by 4000 and symbolically represented by either s or S on the owner’s
execute permission bit. A classic example of an executable file with the SUID permission set is
passwd:

Practical examples:

passwd: The passwd command has SUID permission, allowing any user to change their own password, even if they don’t have write access to the password file. When a user runs passwd, the program executes with the privileges of the root user, allowing it to modify the password file.

**``ls -l /usr/bin/passwd``**
> -rwsr-xr-x 1 root root 63736 jul 27
2018 /usr/bin/passwd

On the other hand, the SGID bit can be set both on files and directories. With files, its behaviour is
equivalent to that of SUID but the privileges are those of the group owner. When set on a
directory, however, it will allow all files created therein to inherit the ownership of the directory’s
group. Like SUID, SGID is symbollically represented by either s or S on the group’s execute
permission bit. Numerically, it is represented by 2000. You can set the SGID on a directory by
using chmod. You have to add 2 (SGID) to the traditional permissions (755 in our case):
example:
>touch file1
chmod g+s file1
to check and veryfy : ls -l file1

To find files with either or both the SUID and SGID set you can use the find command and make
use of the -perm option. You can use both numeric and symbolic values. The values — in
turn — can be passed on their own or preceded by a dash (-) or a forward slash (/). The meaning
is as follows:
* **perm numeric-value or -perm symbolic-value**
find files having the special permission exclusively
* **perm -numeric-value or -perm -symbolic-value**
find files having the special permission and other permissions
* **perm /numeric-value or -perm /symbolic-value**
find files having either of the special permission (and other permissions)

**For example:**
 * To find files with only SUID set in the present working directory, you will use the
following command:
> *numerically:* **find . -perm 4000**
> *symbolically:*  **find . -perm u+s**

* To find files matching SUID (irrespective of any other permssions) in the /usr/bin/ directory,
you can use either of the following commands:
>*numerically :*  sudo find /usr/bin -perm -4000
*symbolically:*  sudo find /usr/bin -perm -u+s

**If you are looking for files in the same directory with the SGID bit set, you can execute find
/usr/bin/ -perm -2000 or find /usr/bin/ -perm -g+s.**
Finally, to find files with either of the two special permissions set and even other permissions, add **4** and **2** and use **/**:

**``find /usr/bin -perm /6000``**

**Output:**
>/usr/bin/fusermount3
/usr/bin/at
/usr/bin/sudo
/usr/bin/chsh
/usr/bin/su
/usr/bin/mount
/usr/bin/newgrp
/usr/bin/passwd
/usr/bin/dotlock.mailutils
/usr/bin/umount
/usr/bin/plocate
/usr/bin/expiry
/usr/bin/crontab
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/chage
/usr/bin/ssh-agent
/usr/bin/pkexec

you could still eneter the command bellow to check it's permission

**``ls -l file name``** 

## Password Management and Aging

As noted above you can use the passwd utility to change your own password as a regular user.
Additionally, you can pass the -S or --status switch to get status information about your
account:
>➜~ passwd -S
maxwell P 09/09/2024 0 99999 7 -1

Here is a breakdown of the seven fields you get in the output:
* maxwell
User’s login name.
* P
It indicates that the user has a valid password (P); other possible values are L for a locked
password and NP for no password.
* 12/07/2019
Date of the last password change.
484
* 0
Minimum age in days (the minimum number of days between password changes). A value of 0
means the password may be changed at any time.
* 99999
Maximum age in days (the maximum number of days the password is valid for). A value of
99999 will disable password expiration.
* 7
Warning period in days (the number of days prior to password expiration that a user will be
warned).   

* -1
Password inactivity period in days (the number of inactive days after password expiration
before the account is locked). A value of -1 will remove an account’s inactivity.


Apart from reporting on account status, you will use the passwd command as root to carry out some basic account maintenance. *You can lock and unlock accounts, force a user to change their
password on the next login and delete a user’s password with the -l, -u, -e and -d options,
respectively*

To test these options it is convenient to introduce the su command at this point. Through su you
can change users during a login session. So, for example, let us use passwd as root to lock maxwell's
password. Then we will switch to maxwell and check our account status to verify that the password
has — in fact — been locked (L) and cannot be changed. Finally, going back to the root user, we
will unlock maxwell's password:

>**output:**
root@debian:~# passwd -l carol
passwd: password expiry information changed.
root@debian:~# su - carol
carol@debian:~$ passwd  -S
carol L 05/31/2020 0 99999 7 -1
carol@debian:~$ passwd
Changing password for carol.
Current password:
passwd: Authentication token manipulation error
passwd: password unchanged
carol@debian:~$ exit
logout
root@debian:~# passwd -u carol
passwd: password expiry information changed.



Alternatively, you can also lock and unlock a user’s password with the **usermod** command:
**Lock user carol's password**
``**usermod -L carol or usermod --lock carol.**``

**Unlock user carol's password**
``**usermod -U carol or usermod --unlock carol.**``

#### NOTE
With the **-f** or **--inactive** switches, usermod can also be used to set the number
of days before an account with an expired password is disabled (e.g.: **usermod -f**
**3 carol**).
Aside from *passwd and usermod*, the most direct command to deal with password and account
aging is **chage (**“change age”**)**. As root, you can pass **chage -l (or --list) username** switch followed by to have that user’s current password and account expiry information printed on the
screen; as a regular user, you can view your own information:

**Run without options and only followed by a username, chage will behave interactively:**

**The options to modify the different chage settings are as follows:**

* **-m days username or --mindays days username**
Specify minimum number of days between password changes (e.g.: chage -m 5 carol). A
value of 0 will enable the user to change his/her password at any time.
* **-M days username or --maxdays days username**
Specify maximum number of days the password will be valid for (e.g.: chage -M 30 carol).
To disable password expiration, it is customary to give this option a value of 99999.
* **-d days username or --lastday days username**
Specify number of days since the password was last changed (e.g.: chage -d 10 carol). A
value of 0 will force the user to change their password on the next login.
* **-W days username or --warndays days username**
Specify number of days the user will be reminded of their password being expired.
* **-I days username or --inactive days username**
Specify number of inactive days after password expiration (e.g.: chage -I 10 carol) — the
same as usermod -f or usermod --inactive. Once that number of days has gone by, the
account will be locked. With a value of 0, the account will not be locked, though.
* **-E date username or --expiredate date username**
Specify date (or number of days since the epoch — January, 1st 1970) on which the account will
be locked. It is normally expressed in the format YYYY-MM-DD(e.g.: chage -E 2050-12-13
carol).
**NOTE**
You can learn more about passwd, usermod and chage — and their options — by
consulting their respective manual pages.

## Limits on Users Logins, Processes and Memory Usage

Resources on a Linux system are not unlimited so — as a sysadmin — you should ensure a good
balance between user limits on resources and the proper functioning of the operating system.
**ulimit** can help you out in that regard.
ulimit deals with soft and hard limits — specified by the **-S** and **-H** options, respectively. Run
without options or arguments, ulimit will display the soft file blocks of the current user:

With the **-a** option, ulimit will show all current soft limits (the same as -Sa); to display all
current hard limits, use -Ha:

The ulimit has switches which could be use in getting and seting user limits

The available shell resources are specified by options such as:
**-b**
maximum socket buffer size
**-f**
maximum size of files written by the shell and its children
**-l**
maximum size that may be locked into memory
**-m**
maximum resident set size (RSS) — the current portion of memory held by a process in main
memory (RAM)
**-v**
maximum amout of virtual memory
**-u**
maximum number of processes available to a single user

Thus, to display limits you will use ulimit followed by either -S (soft) or -H(hard) and the
resource option; if neither -S or -H is supplied, soft limits will be shown:

**example:**
 To get the maximum size of files written by the shell and its children to this user(current user ) 
``ulimit -Sf  ``

**Likewise, to set new limits on a particular resource you will specify either -S or -H, followed by**
**the corresponding resource option and the new value. This value can be a number or the special**
**words soft (current soft limit), hard (current hard limit) or unlimited (no limit). If neither -S or**
**-H is specified, both limits will be set. For example,**
>ulimit -Sf 500

**NOTE**
Hard limits can only be increased by the root user. On the other hand, regular users can decrease
hard limits and increase soft limits up to the value of hard limits. To make new limit values
persistent across reboots, you must write them to the **/etc/security/limits.conf** file. This is
also the file used by the administrator to apply restrictions on particular users.

## Dealing with Logged in Users

Another of your jobs as a sysadmin entails keeping track of logged-in users. There are three
utilities that can help you out with those tasks: **last, who and w**.
**last** prints a listing of the last logged in users with the most recent information on top:
**NOTE**
To check for bad login attempts, run **lastb** instead of **last**.
this can only be done by the root user 
Then you find eight columns; let us break them down:
* **USER**
Login name of user.
* **TTY**
Name of terminal the user is on.
* **FROM**
Remote host from which the user has logged on.
* **LOGIN@**
Login time.
* **IDLE**
Idle time.
* **JCPU**
Time used by all processes attached to the tty (including currently running background jobs).
* **PCPU**
Time used by current process (the one showing under WHAT).
* **WHAT**
**Command line of current process.**
    **who - show who is logged on**

**w - Show who is logged on and what they are doing.**


## Basic sudo Configuration and Usage
As already noted in this lesson, **su** lets you switch to any other user in the system as long as you
provide the target user’s password. In the case of the root user, having its password distributed or
known to (many) users puts the system at risk and is a very bad security practice. The basic usage
of su is su - target-username. When changing to root — though — the target username is
optional:

``su root``
``su - root``
The use of the dash **(-)** ensures that the target user’s environment is loaded. Without it, the old
user’s environment will be kept:
On the other hand, there is the sudo command. With it you can execute a command as the root
user — or any other user for that matter. From a security perspective, sudo is a far better option
than su as it presents two main advantages:
1. to run a command as root, you do not need the root user’s password, but only that of the
invoking user in compliance with a security policy. The default security policy is sudoers as
specified in **/etc/sudoers and /etc/sudoers.d/*.**
2. sudo lets you run single commands with elevated privileges instead of launching a whole new
subshell for root as su does.
The basic usage of sudo is sudo -u target-username command. However, to run a command as
user root, the -u target-username option is not necessary:

 ## The /etc/sudoers File
 Default configuration file for sudo security policy
 >sudo less /etc/sudoers
 
 The privilege specification for the root user is ALL=(ALL:ALL) ALL. This translates as: user root
(root) can log in from all hosts (ALL), as all users and all groups ((ALL:ALL)), and run all
commands (ALL). The same is true for members of the sudo group — note how group names are
identified by a preceding percent sign (%).
if a you want to give a user some more privilages, it won't be advisable to do it manually in the 
configuration file beacuse in the course of modifying mqy couse some errors so it will be better to 
add the user in the sudo group using the command

> sudo usermod aG sudo  <user_name>

Or you could use the *visudo file*  for a more safe environment  
> visudo
if not found then install **sudo**

You may want to save maxwell the inconvenience of having to provide his password to run the
systemctl status apache2 command. For that, you will modify the line to look like this:

> maxwell ALL=(ALL:ALL) NOPASSWD: /usr/bin/systemctl status apache2

Aside from users and groups, you can also make use of aliases in /etc/sudoers. There are three
main categories of aliases that you can define: **host aliases (Host_Alias), user aliases**
**(User_Alias) and command aliases (Cmnd_Alias)**. 



   * **User Aliases**:
    Define a list of users with a single name, making it easier to manage permissions. 
   For example:
   > User_Alias ADMINS = smith, john, wales
   Then, you can use ADMINS instead of listing individual user names throughout the file.
   * **Host Aliases**
    Group multiple hosts together, making it simpler to manage access control. 
   For instance:
   >Host_Alias WEBSERVERS = www1, www2, www3
   This allows you to refer to the WEBSERVERS alias instead of listing each individual host.
   * **Command Aliases:**
   Define a list of commands with a single name, making it easier to manage command execution. 
   For example:
   >Cmnd_Alias REBOOT = /sbin/halt, /sbin/reboot, /sbin/poweroff

Then, you can use **REBOOT**instead of listing individual commands




