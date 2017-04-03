
New system:
    yum install openssh-server
    In VirtualBox, add Port Forwarding for ssh: TCP 2022 -> TCP 22
    yum install epel-release



Video 0302 - Basic Shell Commands

ssh -p 2022 ladmin@127.0.0.1

ls
ls -la
cd /
cd ..
mkdir <directory_name>
pwd



Video 0303 - User and Group Files

cd /etc
cat passwd
    root:x:0:0:root:/root:/bin/bash
    |    | | | |    |     |
    |    | | | |    |      --- Program that runs during interactive login
    |    | | | |    |
    |    | | | |     --------- User's home directory
    |    | | | |
    |    | | |  -------------- User's real "friendly" name
    |    | | |
    |    | |  ---------------- User's primary groupid
    |    | |
    |    |  ------------------ User's userid
    |    |
    |     -------------------- Password is in the shadow file
    |
     ------------------------- User's username

    bin:x:1:1:bin:/bin:/sbin/nologin
        "/sbin/nologin" means you can't login as a given user
        Used for a user associated with a particular service

cat shadow
    root:$6$NekTP...E2jK4F1:17191:0:99999:7: : :
    |    |                  |     | |     | | | |
    |    |                  |     | |     | | |  ---- Reserved for possible future use
    |    |                  |     | |     | | |
    |    |                  |     | |     | |  ------ Number of days since January 1, 1970 that an account has been (will be) disabled
    |    |                  |     | |     | |
    |    |                  |     | |     |  -------- Number of days after password expires that account is disabled (inactive days); change with "passwd -i"
    |    |                  |     | |     |
    |    |                  |     | |      ---------- The number of days to warn user of an expiring password (7 for a full week); change with "passwd -w"
    |    |                  |     | |
    |    |                  |     |  ---------------- Number of days after which password must be changed; change with "passwd -x"
    |    |                  |     |
    |    |                  |      ------------------ Number of days before password may be changed (0: it may be changed at any time); change with "passwd -n"
    |    |                  |
    |    |                   ------------------------ The number of days (since January 1, 1970) since the password was last changed
    |    |
    |     ------------------------------------------- Encrypted password; blank = no password required; "*" = account disabled; "!!" = account locked
    |
     ------------------------------------------------ Username, up to 8 chars; case-sensitive, usually lowercase; matches to username in /etc/passwd

cat group
    ladmin:x:1000:ladmin,kilroy,bob
    |      | |    |
    |      | |     ---- Group members
    |      | |
    |      |  --------- Group's groupid
    |      |
    |       ----------- No password
    |
     ------------------ Group's name



Video 0304 - Creating Users

To become root:
    su
        "switch user"; just become root in the current directory

    su -
        "switch user"; become root with root's home directory/settings

useradd -s /bin/bash -d /home/kilroy -m kilroy
    *ALSO*
        useradd -s /bin/bash -md /home/bilroy bilroy


Video 0305 - Modifying Users

cat /etc/shadow | grep kilroy
    kilroy:!!:17186:0:99999:7:::

usermod -c "This is Kilroy" kilroy

cat /etc/passwd | grep kilroy
    kilroy:x:1001:1001:This is Kilroy:/home/kilroy:/bin/bash

usermod -md /home/new_dir_name kilroy
    /home/new_dir_name

usermod -md /home/kilroy kilroy
    /home/kilroy



Video 0306 - Deleting Users

userdel -r kilroy
    Removes user "kilroy" and its home directory



Video 0307/0308 - Password Policy/Changing Passwords

vi /etc/login.defs

vi /etc/pam.d/system-auth
    password    requisite     pam_pwquality.so try_first_pass local_users_only retry=3 minlen=8 ucredit=-1 lcredit=-1 dcredit=-1 ocredit=-1 authtok_type=

passwd kilroy



Video 0309 - Creating Groups

groupadd mygroup
cat /etc/group

groupdel mygroup
cat /etc/group

groupadd -g 1234 mygroup
cat /etc/group



Video 0310 - Group Membership

usermod -g kilroy kilroy
    cat /etc/passwd
    Sets kilroy's primary group (in /etc/passwd) to kilroy

usermod -G mygroup kilroy
    cat /etc/group
    Sets only mygroup to kilroy's supplemental group (in /etc/group); no impact on primary group

usermod -G mygroup,ladmin kilroy
    cat /etc/group
    Sets both mygroup and ladmin to kilroy's supplemental group (in /etc/group); no impact on primary group

usermod -G wheel -a kilroy
    cat /etc/group
     Adds the wheel group to kilroy's existing supplemental group (in /etc/group); no impact on primary group

All group changes require the user to log out/in again

Setting the umask in /etc/bashrc and /etc/profile
    if [ $UID -gt 199 ] && [ "`/usr/bin/id -gn`" = "`/usr/bin/id -un`" ]; then
        umask 002
    else
        umask 022



Video 0311 - Switching Users

Non-login shell (Applies to Bash target only):
    su
        Runs /root/.bashrc (which sources /etc/bashrc)
    su ladmin
        Runs /home/ladmin/.bashrc (which sources /etc/bashrc)

Login shell (If not a Bash target, only /etc/profile is sourced):
    su -
        Runs /etc/profile then /root/.bash_profile (which sources /root/.bashrc (which sources /etc/bashrc))
            Parts of /etc/bashrc don't run during an interactive shell (e.g. setting umask)
    su - ladmin
        Runs /etc/profile then /home/ladmin/.bash_profile (which sources /home/ladmin/.bashrc (which sources /etc/bashrc))
            Parts of /etc/bashrc don't run during an interactive shell (e.g. setting umask)

Setting the umask in /etc/bashrc and /etc/profile
    if [ $UID -gt 199 ] && [ "`/usr/bin/id -gn`" = "`/usr/bin/id -un`" ]; then
        umask 002
    else
        umask 022

    umask for files masks 666: a umask of 022 results in files with permission 644

    umask for directories masks 777: a umask of 022 results in directories with permission 755



Video 0312 - Using Sudo

sudo cat /etc/passwd

ladmin is not in the sudoers file.  This incident will be reported.
    Reported here: /var/log/secure

Edit the sudoers list
    visudo
    File actually lives here: vi /etc/sudoers

Add users to wheel group, which can perform all actions after password

Become root using sudo

    Non-login shell (Applies to Bash target only):
        sudo -s
            Runs /root/.bashrc (which sources /etc/bashrc)

    Login shell (If not a Bash target, only /etc/profile is sourced):
        sudo -i
            Runs /etc/profile then /root/.bash_profile (which sources /root/.bashrc (which sources /etc/bashrc))
                Parts of /etc/bashrc don't run during an interactive shell (e.g. setting umask)



Video 0313 - Getting Help

man (manual) pages
    'space' or 'f' for next page
    'b' for previous page
    'q' for quit

To find a search term in the man pages: apropos <search_term>
    apropos find

Menu-based information: info
    Scroll over a term then hit Enter
    'd' returns to the main page
    'h' basic info command keys

Sometimes the following flags are available
    <command> -h
    <command> --help



Video 0401 - Redirection

ps auxww > ps.txt
    vi ps.txt

gcc
gcc 2> errors.txt
    '2>' is 'stderr'



Video 0402 - Piping

This is  a pipe: |

ps auxw | grep crond
    root      2376  0.1  0.0 126224  1652 ?        Ss   11:25   0:00 /usr/sbin/crond -n
    root      2399  0.0  0.0 112648   960 pts/0    R+   11:30   0:00 grep --color=auto crond

ps auxw | grep crond | grep -v grep
    root      2376  0.0  0.0 126224  1652 ?        Ss   11:25   0:00 /usr/sbin/crond -n

ps auxw | grep crond | grep -v grep | cut -f 1 -d " "
    root

ps auxw | sort



Video 0403 - Editing

nano
vi



Video 0404 - Regular Expressions

vi ps.txt
    :s/root/r00t/g
        (current line)
    :%s/root/r00t/g
        (whole file)



Video 0405 - Using the Stream Editor

sed (stream editor)

sed 's/root/R00T/g' < ps.txt
    Dumps the updated file contents to stdout (the screen)

sed 's/root/R00T/g' < ps.txt > ps2.txt
    cat ps2.txt

echo "Beef beef Beef" | sed 's/[Bb]eef/BEEF/g'
    BEEF BEEF BEEF



Video 0406 - Using Grep

grep (global replace)

grep ssh ps.txt
    Normal use

grep -i SSH ps.txt
    Case insensitive

grep -n ssh ps.txt
    Include line numbers

grep -R ssh /etc

ps ax | grep bash

ps ax | grep bash | grep -v grep



Video 0407 - File Management

touch
    Alters a file's/folder's access time
    Creates a file if it doesn't already exist

mv
    Renaming files/folders
    Moving files/folders
    Default alias: 'mv -i'

cp
    Copying files/folders
    Default alias: 'cp -i'

rm
    Removing files/folders
    Default alias: 'rm -i'

User 'tab completion'



Video 0408 - Directories

pwd
    Shows current working directory

mkdir tmp
    ls -la
    drwxr-xr-x.  2 root   root       6 Jan 26 12:21 tmp
    ^
    This is a directory

chmod -x tmp
    ls -la
    drw-r--r--.  2 root   root       6 Jan 26 12:23 tmp
       ^  ^  ^
    cd tmp (fails unless root)

chmod +x tmp
    ls -la
    drwxr-xr-x.  2 root   root       6 Jan 26 12:23 tmp
       ^  ^  ^
rmdir tmp

mkdir -p tmp/tmp1/tmp2

rmdir tmp
    Fails because tmp is not empty

rm -Rf tmp



Video 0409 - Permissions

d rwx rwx rwx .
|  |   |   |  |
|  |   |   |   ---- SELinux attributes exist
|  |   |   |
|  |   |    ------- Read, write, execute for all others ('o' for 'others')
|  |   |
|  |    ----------- Read, write, execute for group ('g' for 'group')
|  |
|   --------------- Read, write, execute for owner ('u' for 'user')
|
 ------------------ Items is a directory ('-' for regular file, 'l' for symbolic link)

'r' is worth '4'
'w' is worth '2'
'x' is worth '1'

chmod setting permissions
    777: -rwxrwxrwx.
    755: -rwxr-xr-x.
    700: -rwx------.
    664: -rw-rw-r--. (umask 002)
    644: -rw-r--r--. (umask 022)

chmod add/remove shortcuts:
    Any combination of 'chmod ugo+rwx' or 'chmod ugo-rwx'
    chmod u-r ps.txt   (removes owner's read permission)
    chmod u+r ps.txt   (adds owner's read permission)
    chmod g-w ps.txt   (removes group's write permission)
    chmod g+w ps.txt   (adds group's write permission)
    chmod o-x ps.txt   (removes others' execute permission)
    chmod o+x ps.txt   (adds others' execute permission)
    chmod -r ps.txt    (removes everyone's read permissions)
    chmod +r ps.txt    (adds everyone's read permissions)
    chmod go-w ps.txt  (removes group's and others' write permissions)
    chmod o-rwx ps.txt (removes all permissions from others)



Video 0410 - Using Links

A hard link (ls) makes a file that points to the same data on the disk
    Two different pointers that point to the same place on the disk
    Takes on the date/time of the original file
    Neither file is the "owner" or "original" file now
    A change made to one file affects the content/date/time/permissions of the other
    Either can be deleted without consequence to the other
    Can be created for files only; cannot be created for directories

ln link.txt hardlink.txt
    ls -la
    -rwx------.  2 root   root    8178 Jan 26 12:44 hardlink.txt
    -rwx------.  2 root   root    8178 Jan 26 12:44 link.txt
                 ^
                 Reference count

A symbolic link (ln -s) makes a file that points to the filename of the referenced file
    Gets its own date/time/permissions
    Can be created for directories
    If the link target is deleted, the symbolic link file remains but is 'dead'
    If a new file is created with the target name, the symbolic link 'works' again

ln -s link.txt softlink.txt
    -rwx------.  2 root   root    8178 Jan 26 12:44 link.txt
    lrwxrwxrwx.  1 root   root       6 Jan 26 13:30 softlink.txt -> link.txt
                 ^
                 Reference count



Video 0411 - Archiving & Compressing

tar cvf backup.tar *.txt
    Adds all '.txt' files to the backup.tar archive file
    Files are also preserved in their original location

gzip backup.tar
    Uses gzip to compress backup.tar into backup.tar.gz
    The '.tar' file is transformed; a copy is not made

gunzip backup.tar.gz
    Uses gunzip to decompress backup.tar.gz back to backup.tar
    The '.tar.gz' file is transformed; a copy is not made

tar rvf backup.tar newfile.txt
    Add another file to an existing archive

tar tvf backup.tar
    View the files contained in the archive

tar f backup.tar --delete newfile.txt
    Delete a given file from the archive

tar xvf backup.tar
    All files are extracted from the archive to the current directory
    The original '.tar' file remains unchanged
    Preserves symbolic links; doesn't preserve hard links


tar czvf backup.tar.gz *.txt
    Adds all '.txt' files to the compressed backup.tar.gz archive file in one step
    In this example, the compressed archive is smaller than it was in the two-step process

tar tvf backup.tar.gz
    View the files contained in the compressed archive
    Can't update compressed archives

tar xzvf backup.tar.gz
    All files are extracted from the compressed archive to the current directory
    The original '.tar.gz' file remains unchanged



Video 0412 - Other Utilities

cut -f1 -d "," <filename>
    Default delimited is tab

cat unsorted
    tab     stuff
    foo     wubble
    snow    yuck
    cold    weather
    people  mover
    ...     ...

cut -f1 unsorted
    tab
    foo
    snow
    cold
    people
    ...

cut -f2 unsorted
    stuff
    wubble
    yuck
    weather
    mover
    ...

cut -f1 unsorted | sort
    able
    able
    cold
    cold
    foo
    ...

cut -f1 unsorted | sort | uniq
    able
    cold
    foo
    more
    music
    ...



Video 0413 - Shell History

echo $SHELL
    /bin/bash

cat /home/<user>/.bash_history
cat /root/.bash_history
    Not very up-to-date until logout/login

history
    Up-to-date; pulls commands from memory

!vi
    Runs the last 'vi' command in the history

Ctrl+R
    'reverse-i-search'
    Search backward through the history for a previous command
    Keep pressing Ctrl+R to keep searching backward through the history
    Press Enter to execute the identified command
    Press Ctrl-E to place the command on the CL without executing
    Press Ctrl-C to cancel
    Press Ctrl-Y to pull in whatever is found, but keep searching for other instances
    Press Ctrl-M to execute immediately

Ctrl+S
    Only works after executing 'stty -ixon' to disable 'enable XON/XOFF flow control'
    Need to use Ctrl+R first to move backward through history, then use Ctrl+S to move forward



Video 0414 - Shell Tricks

echo $SHELL
    /bin/bash

Send a process to the background
    <command> &

NONE OF THESE WORK (TODO)
    top &
    fg
    jobs

Conditional execution commands
    gunzip backup.tar.gz && tar xvf backup.tar

Moving quickly through the CL
    Ctrl-A moves the cursor the the beginning of the line
    Ctrl-E moves the cursor the the end of the line



Video 0415 - Locating Files

which ls
    alias ls='ls --color=auto'
    /bin/ls

which find
    /bin/find

find . -name "*.tar.gz" -print
    ./backup.tar.gz

sudo find / -name "*.tar.gz" -print

find . -perm 644



Video 0501 - Booting & Rebooting

Everything else is a child of init; it is the first process

All of the following a sym links to /bin/systemctl
    halt
    poweroff
    reboot
    shutdown

To reboot
    reboot
    systemctl reboot
    shutdown -r now
    shutdown -r +10 "Planned software upgrades"
        Message is only seen when there are max 15 minutes left until shutdown
        To cancel: shutdown -c "Just kidding"
    init 6
    telinit 6

To shutdown
    halt
    systemctl halt
    shutdown -h now
    shutdown -h +30 "Quitting this job"
        Message is only seen when there are max 15 minutes left until shutdown
        To cancel: shutdown -c "Just kidding"
    init 0
    telinit 0

To poweroff
    poweroff
    systemctl poweroff



Video 0502 - Runlevels and Their Uses

vi /etc/inittab

inittab is no longer used when using systemd
    RHEL6/CentOS6 user inittab
    RHEL7/Centos7 use systemd

To view current default target
    systemctl get-default

To set a default target
    systemctl set-default TARGET.target



Video 0503 - Booting into Different Runlevels

Doesn't apply to RHEL7/CentOS7

sudo init 2 (multi-user mode without NFS)
sudo init 3 (multi-user mode)
sudo init 5 (graphical mode)



Video 0504 - Single User Mode

Doesn't apply to RHEL7/CentOS7

sudo init 1 (single-user mode)



Video 0505 - Log Files

cd /var/log

less messages
    These are startup messages

less boot.log
    These are the messages that print to the console during boot

less dmesg
    Kernal and driver related process messages

less secure
    login activity messages are written here
    password modification messages are written here
    useradd/userdel/usermod messages are written here
    groupadd/groupdel/groupmod messages are written here
    ssh messages are written here
    su/sudo messages are written here



Video 0506 - Syslog

cat /etc/rsyslog.conf

"facility" tells syslog the type of message it is
    syslog can send messages to different log files based on facility

"severity" tells syslog the level/severity of the message
    syslog can send messages to different log files based on severity

Any system can be configured to send logs to a remote system via UDP/TCP

Any system can be configured to receive logs from remote systems via UDP/TCP



Video 0507 - Process Management

A process is an instantiation of a program on disk

ps

ps aux
    Cuts off columns at the window size

ps auxww
    Wraps long commands in the terminal window

ps aux | less
    Best method; wraps long commands and is easily scrolled through

top
    List of processes that is updated on a regular basis

Processes are identified by their PID (process ID); signals are sent to processes identified by PID

kill -9 2361
    The "9" is value of the standard signal "SIGKILL"

killall -9 bash

pstree



Video 0508 - Network Services

Didn't really cover network services, more like a system services overview

ls /etc/init.d
    Appears very different on RHEL6/CentOS6 vs RHEL7/CentOS7
    RHEL7/CentOS7 uses systemd instead of init script, though the scripts will still work



Video 0509 - Network Service Management

In RHEL6/CentOS6, the following was standard
    service network status
    service ntpd status

In RHEL7/CentOS7, only 'netconsole' and 'network' are still in /etc/init.d
    'service network status' works
    'service ntpd status' redirects to /bin/systemctl status  ntpd.service (not found)

service --status-all
    Still works, but only on the two services still defined in /etc/init.d

chkconfig
    netconsole      0:off   1:off   2:off   3:off   4:off   5:off   6:off
    network         0:off   1:off   2:on    3:on    4:on    5:on    6:off
        This output shows SysV services only and does not include native systemd services.
        SysV configuration data might be overridden by native systemd configuration.

systemctl list-unit-files
    UNIT FILE                                   STATE
    auditd.service                              enabled
    autovt@.service                             enabled
    blk-availability.service                    disabled
    brandbot.service                            static
    chrony-dnssrv@.service                      static
    chrony-wait.service                         disabled
    chronyd.service                             enabled
    console-getty.service                       disabled
    console-shell.service                       disabled

chkconfig --level 345 ntpd on



Video 0510 - Network Service Management with systemd

Replaces /etc/init.d

cd /etc/systemd

cat /etc/systemd/logind.conf
cat /etc/systemd/system.conf



Video 0511 - Network Service Management with systemctl

Replaces 'services'

systemctl

systemctl -a

systemctl start kdump.service
systemctl stop kdump.service

systemctl daemon-reload



Video 0512 - Package Management

Get a program, its libraries, and documentation

rpm - Red Hat Package Manager

rpm -ivh
    To install a package

yum - Yellowdog Updater, Modified

rpm specifies dependencies; yum automatically installs them first

yum search apache

yum install httpd



Video 0513 - Deleting and Listing Packages

yum list installed

yum erase httpd
    Doesn't remove dependencies that were installed if they can function on their own

yum erase apr
    Removes programs that depend on apr and can't function without it



Video 0514 - Package Details (Location and RPM)

ls /var/lib/rpm

rpm -qf `which pwd`
    The `` executes first and gets replaced within the command

rpm -qdf /bin/pwd
    All documentation associated with the package that contains that command

rpm -qip <some_package_name>.rpm
    View package info



Video 0601 - Partitions

ls /dev
    Every piece of hardware has an entry here

Interested in sda, sdb, sd*
    'sd' stands for 'SCSI disk' even though these aren't SCSI disks
    Interfacing with them like SCSI disks rather than SATA that they actually are

sda, sda1, sda2 means there are two partitions on sda

fdisk -l /dev/sda
    Disk /dev/sda: 8589 MB, 8589934592 bytes, 16777216 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk label type: dos
    Disk identifier: 0x00011aa4

    Device Boot      Start         End      Blocks   Id  System
    /dev/sda1   *        2048     2099199     1048576   83  Linux
    /dev/sda2         2099200    16777215     7339008   8e  Linux LVM

fdisk can be used to create disk partitions

fdisk /dev/sdb
    p
        Disk /dev/sdb: 104 MB, 104851968 bytes, 204789 sectors
        Units = sectors of 1 * 512 = 512 bytes
        Sector size (logical/physical): 512 bytes / 512 bytes
        I/O size (minimum/optimal): 512 bytes / 512 bytes
        Disk label type: dos
        Disk identifier: 0xbc18c853
        Device Boot      Start         End      Blocks   Id  System
    n
        Partition type:
            p   primary (0 primary, 0 extended, 4 free)
            e   extended
                Select (default p): p
                Partition number (1-4, default 1): 1
                First sector (2048-204788, default 2048): 2048
                Last sector, +sectors or +size{K,M,G} (2048-204788, default 204788): 204788
                Partition 1 of type Linux and of size 99 MiB is set
    p
        ...
        Device Boot      Start         End      Blocks   Id  System
        /dev/sdb1            2048      204788      101370+  83  Linux
    w
        The partition table has been altered!
        Calling ioctl() to re-read partition table.
        Syncing disks.

fdisk -l /dev/sdb

Can also use parted to make partitions



Video 0602 - File Systems

After creating a partition, need to put a file system on it

Used to call this 'formatting' meaning putting structure

mkfs.ext2, mkfs.ext3, mkfs.ext4, mkfs.msdos, mkfs.vfat

Create the file system
    mkfs.ext3 /dev/sdb1
    -OR-
    mke2fs -j /dev/sdb1

Need to mount the new file system for it to be accessible
    mount /dev/sdb1 /mnt

ls /mnt
    lost+found



Video 0603 - Volume Management (Physical)

Creating physical volumes allows us to create logical volumes later

Logical volumes are easier to manipulate (add space, shrink space, span volumes, etc.)

umount /dev/sdb1

pvcreate /dev/sdb1
    WARNING: ext3 signature detected on /dev/sdb1 at offset 1080. Wipe it? [y/n]: y
    Wiping ext3 signature on /dev/sdb1.
    Physical volume "/dev/sdb1" successfully created.

    Need a partition or a whole disk for this action

pvdisplay
    --- Physical volume ---
    PV Name               /dev/sda2
    VG Name               cl
    PV Size               7.00 GiB / not usable 3.00 MiB
    Allocatable           yes (but full)
    PE Size               4.00 MiB
    Total PE              1791
    Free PE               0
    Allocated PE          1791
    PV UUID               cN4L3G-Rft4-RXTU-dMfn-vOAU-83Cl-7DYiaE

    "/dev/sdb1" is a new physical volume of "98.99 MiB"
    --- NEW Physical volume ---
    PV Name               /dev/sdb1
    VG Name
    PV Size               98.99 MiB
    Allocatable           NO
    PE Size               0
    Total PE              0
    Free PE               0
    Allocated PE          0
    PV UUID               rZqhnc-1BfF-iuBF-SsEj-T8Za-z1do-lagHCr

pvremove /dev/sdb1
    Labels on physical volume "/dev/sdb1" successfully wiped.

Try to create a physical volume out of the whole disk where a partition exists:
    pvcreate /dev/sdb
        Device /dev/sdb not found (or ignored by filtering).

To create a physical volume out of the whole disk:
    Need to wipe the Master Boot Record (MBR) where the partition table is located
    Can't be a partition table if we want to use the whole disk

    Create a blank MBR, which is the first 512 bytes of any disk
        dd if=/dev/zero of=/dev/sdb bs=512 count=1
            1+0 records in
            1+0 records out
            512 bytes (512 B) copied, 0.000592493 s, 864 kB/s

pvcreate /dev/sdb
    Physical volume "/dev/sdb" successfully created.
pvcreate /dev/sdc
    Physical volume "/dev/sdc" successfully created.
pvcreate /dev/sdd
    Physical volume "/dev/sdd" successfully created.

pvscan
    PV /dev/sda2   VG cl              lvm2 [7.00 GiB / 0    free]
    PV /dev/sdd                       lvm2 [60.00 MiB]
    PV /dev/sdb                       lvm2 [99.99 MiB]
    PV /dev/sdc                       lvm2 [80.00 MiB]
    Total: 4 [7.23 GiB] / in use: 1 [7.00 GiB] / in no VG: 3 [239.99 MiB]



Video 0604 - Volume Management (Logical)

Need physical volumes first to then create logical volumes

Can manipulate size more easily with logical volumes

Can add new disks/partitions into the logical volume

Create a volume group:
    vgcreate myvg /dev/sdb /dev/sdc
        Volume group "myvg" successfully created

Check for the existence of the volume group:
    vgscan
        Reading volume groups from cache.
        Found volume group "cl" using metadata type lvm2
        Found volume group "myvg" using metadata type lvm2

Check for the usage of the physical volumes:
    pvscan
        PV /dev/sda2   VG cl              lvm2 [7.00 GiB / 0    free]
        PV /dev/sdb    VG myvg            lvm2 [96.00 MiB / 96.00 MiB free]
        PV /dev/sdc    VG myvg            lvm2 [76.00 MiB / 76.00 MiB free]
        PV /dev/sdd                       lvm2 [60.00 MiB]
        Total: 4 [7.22 GiB] / in use: 3 [7.16 GiB] / in no VG: 1 [60.00 MiB]

Create a logical volume:
    lvcreate -L 150M myvg
        Rounding up size to full physical extent 152.00 MiB
        Logical volume "lvol0" created.

ls /dev/myvg
    lvol0

Check for the existence of the volume group:
    lvscan
        ACTIVE            '/dev/cl/swap' [820.00 MiB] inherit
        ACTIVE            '/dev/cl/root' [6.20 GiB] inherit
        ACTIVE            '/dev/myvg/lvol0' [152.00 MiB] inherit

fdisk /dev/myvg/lvol0 (don't seem to need to create a partition?)
    p
        Disk /dev/myvg/lvol0: 159 MB, 159383552 bytes, 311296 sectors
        Units = sectors of 1 * 512 = 512 bytes
        Sector size (logical/physical): 512 bytes / 512 bytes
        I/O size (minimum/optimal): 512 bytes / 512 bytes
        Disk label type: dos
        Disk identifier: 0xa9306bc5
        Device Boot      Start         End      Blocks   Id  System
    n
        Partition type:
            p   primary (0 primary, 0 extended, 4 free)
            e   extended
                Select (default p):     p
                Partition number (1-4, default 1):     1
                First sector (2048-311295, default 2048): 2048
                Last sector, +sectors or +size{K,M,G} (2048-311295, default 311295): 311295
                Partition 1 of type Linux and of size 151 MiB is set
    p
        ...
        Device Boot      Start         End      Blocks   Id  System
        /dev/myvg/lvol0p1            2048      311295      154624   83  Linux
    w
    The partition table has been altered!
    Calling ioctl() to re-read partition table.
    WARNING: Re-reading the partition table failed with error 22: Invalid argument.
    The kernel still uses the old table. The new table will be used at
    the next reboot or after you run partprobe(8) or kpartx(8)

Put a filesystem on the new logical volume:
    mke2fs -j /dev/myvg/lvol0

Check the device mapper:
    ls /dev/mapper/myvg-lvol0



Video 0606 - Extending Logical Volumes

Check for available physical volumes
    pvscan
        PV /dev/sda2   VG cl              lvm2 [7.00 GiB / 0    free]
        PV /dev/sdb    VG myvg            lvm2 [96.00 MiB / 0    free]
        PV /dev/sdc    VG myvg            lvm2 [76.00 MiB / 20.00 MiB free]
        PV /dev/sdd                       lvm2 [60.00 MiB]
        Total: 4 [7.22 GiB] / in use: 3 [7.16 GiB] / in no VG: 1 [60.00 MiB]

Add the unallocated physical volume /dev/sdd to the volume group myvg
    vgextend myvg /dev/sdd
        Volume group "myvg" successfully extended

Check the current logical volume size
    lvscan
        ACTIVE            '/dev/cl/swap' [820.00 MiB] inherit
        ACTIVE            '/dev/cl/root' [6.20 GiB] inherit
        ACTIVE            '/dev/myvg/lvol0' [152.00 MiB] inherit

Extend the logical volume lvol0 that is part of volume group myvg
    lvextend -L 162M /dev/myvg/lvol0
        Rounding size to boundary between physical extents: 164.00 MiB.
        Size of logical volume myvg/lvol0 changed from 152.00 MiB (38 extents) to 164.00 MiB (41 extents).
        Logical volume myvg/lvol0 successfully resized.

It used the free space from /dev/sdc before consuming /dev/sdd
    pvscan
        PV /dev/sda2   VG cl              lvm2 [7.00 GiB / 0    free]
        PV /dev/sdb    VG myvg            lvm2 [96.00 MiB / 0    free]
        PV /dev/sdc    VG myvg            lvm2 [76.00 MiB / 8.00 MiB free]
        PV /dev/sdd    VG myvg            lvm2 [56.00 MiB / 56.00 MiB free]
        Total: 4 [7.22 GiB] / in use: 4 [7.22 GiB] / in no VG: 0 [0   ]



Video 0605 - Mounting Remote Volumes

mount -t cifs -o username=myuser,password=mypassword //10.0.0.14/temp /temp

mount
    //10.0.0.14/temp on /temp type cifs (rw)

Edit /etc/fstab to make the mount survive reboot
    //10.0.0.14/temp    /temp   cifs    user,rw,username=myuser,password=mypassword 0   0



Video 0416 - Extending ext4 Partitions

fdisk /dev/sdd
    d
    w

cfdisk /dev/sdd
    [New]
    [Primary]
    Size (in MB): 40
    [Beginning]
    [Write]
    Are you sure you want to write the partition table to disk? (yes or no): yes
    q

    Disk has been changed.

mkfs.ext4 /dev/sdd1

mount /dev/sdd1 /mnt

touch /mnt/file1
touch /mnt/file2

ls -la /mnt
    -rw-r--r--.  1 root root     0 Jan 31 05:37 file1
    -rw-r--r--.  1 root root     0 Jan 31 05:37 file2
    drwx------.  2 root root 12288 Jan 31 05:36 lost+found

umount /mnt

Make the partition larger:
    cfdisk /dev/sdd
        [Delete]
        [New]
        [Primary]
        Size (in MB): 62.91
        [Write]
        Are you sure you want to write the partition table to disk? (yes or no): yes
        q

The partition table's end point was changed, but no data was altered.

Extend the file system:
    resize2fs -f /dev/sdd1
        resize2fs 1.42.9 (28-Dec-2013)
        Resizing the filesystem on /dev/sdd1 to 61404 (1k) blocks.
        The filesystem on /dev/sdd1 is now 61404 blocks long.

mount /dev/sdd1 /mnt

ls -la /mnt
    -rw-r--r--.  1 root root     0 Jan 31 05:37 file1
    -rw-r--r--.  1 root root     0 Jan 31 05:37 file2
    drwx------.  2 root root 12288 Jan 31 05:36 lost+found



Video 0607 - Using LUKS for Encryption

Linux Unified Key Setup (LUKS)

Encrypt a disk:
    cryptsetup luksFormat /dev/sdd
        WARNING!
        ========
        This will overwrite data on /dev/sdd irrevocably.
        Are you sure? (Type uppercase yes): YES
        Enter passphrase:
        Verify passphrase:

Set up the device mapper:
    cryptsetup luksOpen /dev/sdd mycrypt
        Enter passphrase for /dev/sdd:

Check the device mapper:
    ls /dev/mapper/mycrypt

Want to mount this when we boot the system, so we create a key:
    dd if=/dev/urandom of=/root/keyfile bs=1024 count=4
        4+0 records in
        4+0 records out
        4096 bytes (4.1 kB) copied, 0.000400846 s, 10.2 MB/s

ls /root/keyfile

Add the keyfile to the encrypted volume's keychain:
    cryptsetup luksAddKey /dev/sdd /root/keyfile
        Enter any existing passphrase:

Check the device status:
    cryptsetup status mycrypt
        /dev/mapper/mycrypt is active.
        type:    LUKS1
        cipher:  aes-xts-plain64
        keysize: 256 bits
        device:  /dev/sdd
        offset:  4096 sectors
        size:    118781 sectors
        mode:    read/write

Format the encrypted volume, not the underlying device:
    mkfs.ext4 /dev/mapper/mycrypt

Mount the device:
    mkdir /crypt
    mount /dev/mapper/mycrypt /crypt

Create an entry in /etc/fstab to automount the volume:
    vi /etc/fstab
    /dev/mapper/mycrypt /crypt  ext4    defaults    1   2

Associate a key so the volume will be decrypted and automounted on boot:
    vi /etc/crypttab
    mycrypt /dev/sdd    /root/keyfile   luks

Close the encrypted volume:
    umount /crypt
    cryptsetup luksClose mycrypt



Video 0608 - Using SetGID

Like the setUID bit (doesn't seem to apply on RHEL7/CentOS7)
    ls -la `which ping`
        /bin/ping (root)
        /usr/bin/ping (ladmin)

By default, if a user has group write permissions in a directory it doesn't own,
any file/directory it creates will inherit the primary GID of the user that creates it.

Setting the GID with 'chmod g+s' will change the behavior such that any file/directory
created by a user authorized to write in the directory will inherit the directory's GID.

As ladmin:
    ls -lad /opt/ladmin
        drwxrwxr-x. 2 ladmin ladmin 25 Jan 31 02:14 /opt/ladmin
              ^              ^^^^^^

    touch /opt/ladmin/ladmin_file

    ls -la /opt/ladmin
        -rw-rw-r--. 1 ladmin ladmin   0 Jan 31 02:20 ladmin_file
                             ^^^^^^

As kilroy (need to be in the ladmin group):
    touch /opt/ladmin/kilroy_file

    ls -la /opt/ladmin
        -rw-rw-r--. 1 kilroy kilroy   0 Jan 31 02:28 kilroy_file
                             ^^^^^^

As ladmin:
    chmod g+s /opt/ladmin

    ls -lad /opt/ladmin
        drwxrwsr-x. 2 ladmin ladmin 25 Jan 31 02:14 /opt/ladmin
              ^              ^^^^^^

As kilroy (need to be in the ladmin group):
    touch /opt/ladmin/kilroy_file2

    ls -la /opt/ladmin
        -rw-rw-r--. 1 kilroy ladmin   0 Jan 31 02:33 kilroy_file2
                             ^^^^^^

This allows anyone in the shared group to access any files/folders created
in the directory, regardless of who actually created the file. Otherwise,
a user may create a file with a default GID that no one else is able to access.



Video 0609 - Access Control Lists (ACLs)

mount -t ext4 -o acl /dev/sdd1 /mnt
    Don't seem to need to set '-o acl' option on RHEL7/CentOS7

mount
    /dev/sdd1 on /mnt type ext4 (rw,relatime,seclabel,data=ordered)

touch foo

mkdir kilroy

setfacl -m u:kilroy:rwx /mnt/kilroy

ls -lad /mnt/kilroy
    drwxrwxr-x+ 2 root root 1024 Jan 31 03:47 /mnt/kilroy
              ^

getfacl /mnt/kilroy
    getfacl: Removing leading '/' from absolute path names
    # file: mnt/kilroy
    # owner: root
    # group: root
    user::rwx
    user:kilroy:rwx
    group::r-x
    mask::rwx
    other::r-x

As kilroy:
    touch /mnt/kilroy/newfile

    ls -la ./newfile
        -rw-rw-r--. 1 kilroy kilroy 0 Jan 31 03:53 ./newfile

setfacl -x u:kilroy /mnt/kilroy

ls -lad /mnt/kilroy
    drwxr-xr-x+ 3 root root 1024 Jan 31 03:54 /mnt/kilroy
              ^

getfacl /mnt/kilroy
    getfacl: Removing leading '/' from absolute path names
    # file: mnt/kilroy
    # owner: root
    # group: root
    user::rwx
    group::r-x
    mask::r-x
    other::r-x



Video 0610 - Permissions Problems

As root:
    touch /home/kilroy/root_was_here

As kilroy:
    ls -la /home/kilroy/root_was_here
        -rw-r--r--. 1 root root 0 Jan 31 04:10 /home/kilroy/root_was_here

    rm /home/kilroy/root_was_here
        rm: remove write-protected regular empty file '/home/kilroy/root_was_here'? y

    Allowed to remove a non-owned file if the directory is owned.

    Can also view and edit, but editing and saving will change owner/group to kilroy.



Video 0611 - Adding Partitions and Volumes

mount /dev/sdd1 /mnt

mount
    /dev/sdd1 on /mnt type ext4 (rw,relatime,seclabel,data=ordered)



Video 0612 - Using Swap Space

Swap space is disk space temporarily used to hold data instead of memory.
Allows us to run more applications that we would using memory alone.
Exists because disk space is usually cheaper than memory.

When an application is idle, it can swap some of the memory out to the disk.

Swaps currently in place:
    cat /proc/swaps
        Filename    Type        Size    Used    Priority
        /dev/dm-1   partition   839676  0       -1

fdisk /dev/sdd
    Command (m for help): n
        Partition type:
            p   primary (0 primary, 0 extended, 4 free)
            e   extended
        Select (default p): p
        Partition number (1-4, default 1): 1
        First sector (2048-122876, default 2048): 2048
        Last sector, +sectors or +size{K,M,G} (2048-122876, default 122876): +50M
        Partition 1 of type Linux and of size 50 MiB is set
    Command (m for help): t
        Selected partition 1
        Hex code (type L to list all codes): 82
        Changed type of partition 'Linux' to 'Linux swap / Solaris'
    Command (m for help): p
        Disk /dev/sdd: 62 MB, 62913024 bytes, 122877 sectors
        Units = sectors of 1 * 512 = 512 bytes
        Sector size (logical/physical): 512 bytes / 512 bytes
        I/O size (minimum/optimal): 512 bytes / 512 bytes
        Disk label type: dos
        Disk identifier: 0xbc371a4c
            Device Boot     Start   End     Blocks  Id  System
            /dev/sdd1       2048    104447  51200   82  Linux swap / Solaris
    Command (m for help): w
        The partition table has been altered!
        Calling ioctl() to re-read partition table.

mkswap /dev/sdd1
    mkswap: /dev/sdd1: warning: wiping old ext4 signature.
    Setting up swapspace version 1, size = 51196 KiB
    no label, UUID=0b5b651f-ad9b-49da-a52f-b63a9b7f6430

swapon /dev/sdd1

cat /proc/swaps
    Filename    Type        Size    Used    Priority
    /dev/dm-1   partition   839676  0       -1
    /dev/sdd1   partition   51196   0       -2

free -g
            total        used        free      shared  buff/cache   available
    Mem:        1           0           1           0           0           1
    Swap:       0           0           0

vmstat
    procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
     r  b  swpd    free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
     1  0     0 1361780    952 397464    0    0    14    34   15   25  0  0 99  1  0

Would have to edit /etc/fstab to make this permanent.

vi /etc/fstab
    tmpsf   /dev/sdd1   tmpfs   defaults    0   0

swapoff /dev/sdd1

cat /proc/swaps
    Filename    Type        Size    Used    Priority
    /dev/dm-1   partition   839676  0       -1



Video 0613 - Booting a Disk Using UUID

Universally Unique Identifier (UUID)

Multiple ways to display the UUID:

lsblk -f
    NAME         FSTYPE      LABEL UUID                                   MOUNTPOINT
    sda
    ├─sda1       xfs               d8610cb6-50b9-42ec-81a8-28d0af45184c   /boot
    └─sda2       LVM2_member       cN4L3G-Rft4-RXTU-dMfn-vOAU-83Cl-7DYiaE
      ├─cl-root  xfs               d94485e4-35ac-4eaa-b9d4-f72d71254fe1   /
      └─cl-swap  swap              b89a53ae-9188-46ad-93fa-c1c7a9cc9889   [SWAP]
    sdb          LVM2_member       MEnUmS-OyR9-aLKu-sA58-9k4Z-5Qgg-y2nYFM
    └─myvg-lvol0 ext3              97f063d1-278d-4e12-962a-bca6dc705c9a
    sdc          LVM2_member       IAMoBe-dtcc-Lt19-iPsb-1oKU-vJkv-30W78I
    └─myvg-lvol0 ext3              97f063d1-278d-4e12-962a-bca6dc705c9a
    sdd
    └─sdd1       swap              0b5b651f-ad9b-49da-a52f-b63a9b7f6430
    sr0

blkid
    /dev/sda1: UUID="d8610cb6-50b9-42ec-81a8-28d0af45184c" TYPE="xfs"
    /dev/sda2: UUID="cN4L3G-Rft4-RXTU-dMfn-vOAU-83Cl-7DYiaE" TYPE="LVM2_member"
    /dev/sdb: UUID="MEnUmS-OyR9-aLKu-sA58-9k4Z-5Qgg-y2nYFM" TYPE="LVM2_member"
    /dev/sdc: UUID="IAMoBe-dtcc-Lt19-iPsb-1oKU-vJkv-30W78I" TYPE="LVM2_member"
    /dev/sdd1: UUID="0b5b651f-ad9b-49da-a52f-b63a9b7f6430" TYPE="swap"
    /dev/mapper/cl-root: UUID="d94485e4-35ac-4eaa-b9d4-f72d71254fe1" TYPE="xfs"
    /dev/mapper/cl-swap: UUID="b89a53ae-9188-46ad-93fa-c1c7a9cc9889" TYPE="swap"
    /dev/mapper/myvg-lvol0: UUID="97f063d1-278d-4e12-962a-bca6dc705c9a" SEC_TYPE="ext2" TYPE="ext3"

ls -la /dev/disk/by-uuid
    lrwxrwxrwx. 1 root root  10 Jan 31 05:03 0b5b651f-ad9b-49da-a52f-b63a9b7f6430 -> ../../sdd1
    lrwxrwxrwx. 1 root root  10 Jan 30 22:13 97f063d1-278d-4e12-962a-bca6dc705c9a -> ../../dm-2
    lrwxrwxrwx. 1 root root  10 Jan 30 22:13 b89a53ae-9188-46ad-93fa-c1c7a9cc9889 -> ../../dm-1
    lrwxrwxrwx. 1 root root  10 Jan 30 22:13 d8610cb6-50b9-42ec-81a8-28d0af45184c -> ../../sda1
    lrwxrwxrwx. 1 root root  10 Jan 30 22:13 d94485e4-35ac-4eaa-b9d4-f72d71254fe1 -> ../../dm-0

cat /etc/fstab | grep boot
    UUID=d8610cb6-50b9-42ec-81a8-28d0af45184c   /boot     xfs     defaults    0 0

vi /boot/grub2/grub.cfg



Video 0701 - Configuring Networking

RHEL6/CentOS6:
    ifconfig -a

RHEL7/CentOS7:
    ip addr

cd /etc/sysconfig/network-scripts
vi ifcfg-enp0s3



Video 0702 - Configuring DNS Resolution

TODO Doesn't work on RHEL7/CentOS7
    host infiniteskills.com

TODO Search Domains

vi /etc/resolv.conf
    Set domain
    Set search
    Set nameserver



Video 0703 - Using Time Services

Clocks on computers have a tendency to drift

RHEL6/CentOS6:
    Edit time servers:
        vi /etc/ntp.conf
    Sync the local clock with remote time source(s):
        /etc/init.d/ntpdate start
    Set local system as a time source:
        /etc/init.d/ntp start

RHEL7/CentOS7:
    Edit time servers:
        vi /etc/chrony.conf
    TODO



Video 0704- Using cron to Set Up Jobs

cd /etc/cron.d

cat /etc/crontab
    SHELL=/bin/bash
    PATH=/sbin:/bin:/usr/sbin:/usr/bin
    MAILTO=root

    # For details see man 4 crontabs

    # Example of job definition:
    # .---------------- minute (0 - 59)
    # |  .------------- hour (0 - 23)
    # |  |  .---------- day of month (1 - 31)
    # |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
    # |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
    # |  |  |  |  |
    # *  *  *  *  * user-name  command to be executed

cat /etc/cron.d/0hourly
    # Run the hourly jobs
    SHELL=/bin/bash
    PATH=/sbin:/bin:/usr/sbin:/usr/bin
    MAILTO=root
    01 * * * * root run-parts /etc/cron.hourly

cat /etc/cron.hourly/0anacron

cd /etc/cron.weekly
    vi webcopy
    mkdir /home/kilroy/scratch
    echo "Hello there world" > /home/kilroy/scratch/index.html



Video 0705 - Installing HTTP

yum search httpd
yum search php

yum install httpd php

ls /var/www

cd /etc/httpd

RHEL6/CentOS6:
    /etc/init.d/httpd start
    chkconfig httpd on

RHEL7/CentOS7:
    systemctl start httpd
    systemctl enable httpd



Video 0706 - Installing FTP

yum search ftp

yum install vsftpd

cd /etc/vsftpd

vi vsftpd.conf
    anonymous_enable=NO
    :wq

systemctl start vsftpd

ftp localhost
    kilroy
    <password>

ftp> ls
ftp> quit



Video 0707 - Configuring Services

cd /etc/httpd/conf
    vi httpd.conf

cd /etc/httpd/conf.d/

cd /etc/httpd/conf.modules.d

Restart is changes are made:
    systemctl restart httpd

TODO Many config differences between httpd on RHEL6/CentOS6 and RHEL7/CentOS7



Video 0708 - Setting Service to Run at Startup

In RHEL6/CentOS6, this was controlled by rc*:
    ls /etc/rc*

    ls -la /etc/rc3.d/
        lrwxrwxrwx.  1 root root  20 Jan 20 12:50 K50netconsole -> ../init.d/netconsole
        lrwxrwxrwx.  1 root root  17 Jan 20 12:50 S10network -> ../init.d/network

    chkconfig

    chkconfig --level 345 vsftpd on


In RHEL7/CentOS7, this is controlled by
    systemctl enable vsftpd.service
        Created symlink from /etc/systemd/system/multi-user.target.wants/vsftpd.service
        to /usr/lib/systemd/system/vsftpd.service



Video 0709 - Using LDAP Server for User Management

In the example, performed via GUI
    System -> Administrator -> Authentication
    User Account Database: Change from "Local accounts only" to "LDAP"
    LDAP Search Base DN ("Distinguished Name")
        "Organizational Unit" (OU); "Domain Context" (DC)
        ou=staff,dc=infiniteskills,dc=com
        LDAP Server: ldaps://windows.infiniteskills.com



Video 0710 - Updating Packages

yum update

Usually won't require a reboot



Video 0711 - Red Hat Repositories

less /etc/yum.conf

cd /etc/yum.repos.d

less CentOS-Base.repo
    Online mirror list

less CentOS-Media.repo
    Use a mounted DVD as a repo

Set up local repos in /etc/yum.repos.d

cd /etc/yum
    vi pluginconf.d/fastestmirror.conf

find / -type f -name "timedhosts.txt"



Video 0712 - Using Kickstart to Deploy Systems

cd /root

less anaconda-ks.cfg
    This is a Kickstart file based on info we provided when doing this install
    Can use a ks file to install other systems with similar hardware

Could add a post-installation step

Either place the anaconda-ks.cfg file:
    - on a disc
    - on a modified ISO
    - using DHCP server to specify a network boot (PXE boot, TFTP boot)



Video 0713 - Manage and Update Kernel

The kernel is basically the operating system:
    - it interfaces with the hardware
    - provides functionality, allowing input from keyboard/mouse
    - process management
    - memory management
    - disk management

yum search kernel

yum update kernel

ls /boot
    Multiple kernels now

If you want to build modules for the kernel; sometimes need kernel drivers
    yum search kernel-devel



Video 0714 - Manage the Boot Loader

RHEL6/CentOS6
    cd /boot/grub
    vi menu.lst

(hd0,1)



Video 0715 - Manage the Boot Loader with Grub2

RHEL7/CentOS7
    cd /boot/grub2
    cd /etc/grub.d
    vi /etc/default/grub
    vi /etc/sysconfig/grub (sym link to the 'default')

grub2-mkconfig -o /boot/grub2/grub.cfg
    Generating grub configuration file ...
    Found linux image: /boot/vmlinuz-3.10.0-514.6.1.el7.x86_64
    Found initrd image: /boot/initramfs-3.10.0-514.6.1.el7.x86_64.img
    Found linux image: /boot/vmlinuz-3.10.0-514.el7.x86_64
    Found initrd image: /boot/initramfs-3.10.0-514.el7.x86_64.img
    Found linux image: /boot/vmlinuz-0-rescue-abc168aa00fe41a6860d1781c3611a4f
    Found initrd image: /boot/initramfs-0-rescue-abc168aa00fe41a6860d1781c3611a4f.img

(hd0,gpt1)
(hd0,msdos1)

grub2-install /dev/sda
    Installing for i386-pc platform.
    Installation finished. No error reported.



Video 0716 - Connecting to Remote Systems Using SSH

ssh 10.0.0.16

ssh -l kilroy 10.0.0.16

ssh kilroy@10.0.0.16

ssh kilroy@10.0.0.16 -p 22

cd ~/.ssh
    vi known_hosts
    127.0.0.1 ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBCEHjxX/ZAM4hsoMOMYI4Ivb3KWeo0MTofZR8r5pLhE77HbFdtxeTqtUalUGnxBjcKDjvndQvxK6H979a0QqdG8



Video 0717 - Using Keys for Logging Into Systems Over SSH

ssh-keygen -t rsa -b 4096
    Generating public/private rsa key pair.
    Enter file in which to save the key (/home/kilroy/.ssh/id_rsa):
    Created directory '/home/kilroy/.ssh'.
    Enter passphrase (empty for no passphrase):
    Enter same passphrase again:
    Your identification has been saved in /home/kilroy/.ssh/id_rsa.
    Your public key has been saved in /home/kilroy/.ssh/id_rsa.pub.
    The key fingerprint is:
    33:46:f7:6b:d3:ff:e5:8f:7b:32:f9:95:36:df:e5:2a kilroy@thor.local
    The key's randomart image is:
    +--[ RSA 4096]----+
    |                 |
    |                 |
    |        . .      |
    |       . . .     |
    |        S   .    |
    |       . o   o  .|
    |            + .=+|
    |           .E.==O|
    |             .+B%|
    +-----------------+

ssh-copy-id kilroy@127.0.0.1

ssh kilroy@127.0.0.1



Video 0718 - Kernel Configuration Using Sysctl

sysctl -a
    abi.vsyscall32 = 1
    crypto.fips_enabled = 0
    debug.exception-trace = 1
    debug.kprobes-optimization = 1
    dev.cdrom.autoclose = 1
    dev.cdrom.autoeject = 0
    dev.cdrom.check_media = 0
    dev.cdrom.debug = 0
    ...

sysctl -w net.ipv4.tcp_timestamps=0
    Set a value on-the-fly; doesn't survive reboot

sysctl net.ipv4.tcp_timestamps
    View a value

RHEL6/CentOS6:
    vi /etc/sysctl.conf

RHEL7/CentOS7:
    cd /usr/lib/sysctl.d
        vi 00-system.conf
        vi 50-default.conf

    cd /etc/sysctl.d
        vi 99-sysctl.conf
        Add '90-override.conf' for settings overrides

sysctl net.ipv4.ip_forward
    net.ipv4.ip_forward = 0

cat /proc/sys/net/ipv4/ip_forward
    0

echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf

sysctl -p /etc/sysctl.conf

sysctl net.ipv4.ip_forward



Video 0801 - Preparing for Using Virtual Machines

Look for Intel processor extensions (vmx) or AMD processor extensions (svm):
    grep '{vmx|svm}' /proc/cpuinfo
        If running on bare metal, there should be results
        This is required for 'Hardware Virtualization'
        If no results, can still do 'Paravirtualization'

yum install kvm kmod-kvm qemu



Video 0802 - Installing RHEL as a Virtual Guest

qemu-img create -f qcow2 CentOS-6.8-hd.img 2G
    Formatting 'mine.img', fmt=qcow2 size=1073741824 encryption=off cluster_size=65536 lazy_refcounts=off

#qemu-kvm -cdrom /root/CentOS-6.8-x86_64-minimal.iso -m 512 -boot d
qemu-system-x86_64 -boot d -cdrom /root/CentOS-6.8-x86_64-minimal.iso -m 512 -hda /root/CentOS-6.8-hd.img -no-acpi

#ON KALI
apt-get install xboot
apt-get install qemu
apt-get install kvm
apt-get install qemu-kvm
apt-get install virt-manager
apt-get install bridge-utils
apt-get install libvirt-bin
egrep -c ‘(svm|vmx)’ /proc/cpuinfo
qemu-system-x86_64 -boot d -cdrom /kali/kali.iso -m 512 -usbdevice host:05dc:a838
info usbhost
lsusb | grep Lexar



Video 0803 - Launch Virtual Machines at Boot

yum search virsh

yum install fence-agents-virsh

virsh
    Welcome to virsh, the virtualization interactive terminal.
    virsh # help

    Used to start VMs at boot

    virsh # list

    virsh # autostart foo

Also need to be running libvirtd

yum install libvirt

systemctl start libvirtd

systemctl enable libvirtd



Video 0804 - Connecting to a Virtual Machine Console

virsh
    virsh # console

Via GUI:
    Virtual Machine Manager
        Can install VMs and connect to the Console



Video 0901 - IPTables

RHEL6/CentOS6:
    iptables -L -v

# Switched to Sander's book

