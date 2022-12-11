# LBFH-notes
Linux basics for hackers - notes

![book cover](./assets/cover.jpg)

## page 4 - the Linux filesystem

On Kali Linux:
* `/bin` is a symbolic link to `/usr/bin`
* `/sbin` is a symbolic link to `/usr/sbin`
* `/lib` is a symbolic link to `/usr/lib`

"things only got moved in to /usr because of storage constraints on ancient Unix machines" ([src](https://unix.stackexchange.com/questions/266517/why-is-bin-a-symbolic-link-to-usr-bin)).

Origin story ([src](http://lists.busybox.net/pipermail/busybox/2010-December/074114.html)):

```
You know how Ken Thompson and Dennis Ritchie created Unix on a PDP-7 in 1969?  
Well around 1971 they upgraded to a PDP-11 with a pair of RK05 disk packs (1.5 
megabytes each) for storage.

When the operating system grew too big to fit on the first RK05 disk pack (their 
root filesystem) they let it leak into the second one, which is where all the 
user home directories lived (which is why the mount was called /usr).  They 
replicated all the OS directories under there (/bin, /sbin, /lib, /tmp...) and 
wrote files to those new directories because their original disk was out of 
space.  When they got a third disk, they mounted it on /home and relocated all 
the user directories to there so the OS could consume all the space on both 
disks and grow to THREE WHOLE MEGABYTES (ooooh!).

Of course they made rules about "when the system first boots, it has to come up 
enough to be able to mount the second disk on /usr, so don't put things like 
the mount command /usr/bin or we'll have a chicken and egg problem bringing 
the system up."  Fairly straightforward.  Also fairly specific to v6 unix of 35 
years ago.

The /bin vs /usr/bin split (and all the others) is an artifact of this, a 
1970's implementation detail that got carried forward for decades by 
bureaucrats who never question _why_ they're doing things.  It stopped making 
any sense before Linux was ever invented, for multiple reasons:

1) Early system bringup is the provice of initrd and initramfs, which deals 
with the "this file is needed before that file" issues.  We've already _got_ a 
temporary system that boots the main system.

2) shared libraries (introduced by the Berkeley guys) prevent you from 
independently upgrading the /lib and /usr/bin parts.  They two partitions have 
to _match_ or they won't work.  This wasn't the case in 1974, back then they 
had a certain level of independence because everything was statically linked.

3) Cheap retail hard drives passed the 100 megabyte mark around 1990, and 
partition resizing software showed up somewhere around there (partition magic 
3.0 shipped in 1997).

Of course once the split existed, some people made other rules to justify it.  
Root was for the OS stuff you got from upstream and /usr was for your site-
local files.  Then / was for the stuff you got from AT&T and /usr was for the 
stuff that your distro like IBM AIX or Dec Ultrix or SGI Irix added to it, and 
/usr/local was for your specific installation's files.  Then somebody decided 
/usr/local wasn't a good place to install new packages, so let's add /opt!  
I'm still waiting for /opt/local to show up...

Of course given 30 years to fester, this split made some interesting distro-
specific rules show up and go away again, such as "/tmp is cleared between 
reboots but /usr/tmp isn't".  (Of course on Ubuntu /usr/tmp doesn't exist and 
on Gentoo /usr/tmp is a symlink to /var/tmp which now has the "not cleared 
between reboots" rule.  Yes all this predated tmpfs.  It has to do with read-
only root filesystems, /usr is always going to be read only in that case and 
/var is where your writable space is, / is _mostly_ read only except for bits 
of /etc which they tried to move to /var but really symlinking /etc to 
/var/etc happens more often than not...)

Standards bureaucracies like the Linux Foundation (which consumed the Free 
Standards Group in its' ever-growing accretion disk years ago) happily 
document and add to this sort of complexity without ever trying to understand 
why it was there in the first place.  'Ken and Dennis leaked their OS into the 
equivalent of home because an RK05 disk pack on the PDP-11 was too small" goes 
whoosh over their heads.
```

On Kali Linux, the `mount` command can be located is the `/usr/bin` directory, without the "chicken and egg" problem, because the `/usr` directory is not a mount point any more. Disks are large enough to include the whole OS on the boot filesystem.

Other explanation: [https://askubuntu.com/questions/130186/what-is-the-rationale-for-the-usr-directory](https://askubuntu.com/questions/130186/what-is-the-rationale-for-the-usr-directory)

### Filesystem Hierarchy Standard and Kali

[https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard](https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard)

* `/usr`: all system-wide (shareable), read-only user data; contains the majority of (multi-)user utilities and applications
---
* `/bin → /usr/bin`: command binaries
* `/sbin → /usr/sbin`: "Essential system binaries". Commands that can only be (or are only meaningful when) executed by the root user, like mount, fdisk, [daemons](https://en.wikipedia.org/wiki/Daemon_(computing)) for various [network services](https://en.wikipedia.org/wiki/Network_service)
* `/lib → /usr/lib`: Libraries for the binaries in `/usr/bin` and `/usr/sbin`
* `/usr/include`: Standard [include files](https://en.wikipedia.org/wiki/Include_directive)
* `/usr/share`: Architecture-independent (shared) data

Example:
```
└─$ whereis aircrack-ng
aircrack-ng: /usr/bin/aircrack-ng /usr/include/aircrack-ng /usr/share/man/man1/aircrack-ng.1.gz
```

---
* `/boot`: Boot loader files (kernel image, initrd)
* `/proc`: Virtual filesystem providing process and kernel information as files. In Linux, corresponds to a procfs mount. Generally, automatically generated and populated by the system, on the fly. 
* `/sys`: Kernel's view of the hardware. Contains information about devices, drivers, and some kernel features
---
* `/opt`: "Add-on application software packages". An atrocity meant for system-wide, read-only and self-contained software. That is, software that does not split their files over `bin`, `lib`, `share`, `include` like well-behaved software should
---
* `/etc`: Host-specific system-wide configuration files. Files that control when and how programs start up. (In early versions of the UNIX Implementation Document from Bell labs, /etc is referred to as the etcetera directory, as this directory historically held everything that did not belong elsewhere (however, the FHS restricts /etc to static configuration files and may not contain binaries). Since the publication of early documentation, the directory name has been re-explained in various ways. Recent interpretations include backronyms such as "Editable Text Configuration" or "Extended Tool Chest")
---
* `/var`: Variable files: files whose content is expected to continually change during normal operation of the system, such as logs, spool files, and temporary e-mail files. 
* `/var/log`: Log files. Various logs. 
* `/var/lib`: State information. Persistent data modified by programs as they run (e.g., databases, packaging system metadata, etc.). 
---
* `/dev`: directory contains special files (device files) corresponding to physical devices or system components, as things you wouldn't normally think of as devices such as [/dev/null](https://en.wikipedia.org/wiki/Null_device).
* `/media`: "Mount points for removable media", where CDs and USB devices are usually mounted to the filesystems
* `/mnt`: "Temporarily mounted filesystems". Used to mount other filesystems, usually for a short period of time

[https://unix.stackexchange.com/questions/309706/what-is-the-difference-between-dev-media-and-mnt](https://unix.stackexchange.com/questions/309706/what-is-the-difference-between-dev-media-and-mnt)

[https://unix.stackexchange.com/questions/13975/mounting-a-device-role-of-dev-media-and-mnt-and-the-mount-command](https://unix.stackexchange.com/questions/13975/mounting-a-device-role-of-dev-media-and-mnt-and-the-mount-command)

---
* `/root`: Home directory for the root user. 
* `/home`: Home directories for other users. Containing saved files, personal settings, etc. 
* `/tmp`: Directory for temporary files (see also /var/tmp). Often not preserved between system reboots and may be severely size-restricted. 

## page 23 - challenge using `nl` and `grep`

```
sudo nl /etc/snort/snort.conf | grep -B 5 '# Step #6: Configure'
```

## page34 - dig

### A
get the IP addresses of cat-amania.com:
```
dig cat-amania.com A +noall +answer
```

### MX
get the mailservers of cat-amania.com:
```
dig cat-amania.com MX +noall +answer
```

### NS
get a list of authoritative DNS servers for cat-amania.com:
```
dig cat-amania.com NS +noall +answer
```

This command returns:
```
cat-amania.com.         10800   IN      NS      ns-146-a.gandi.net.
cat-amania.com.         10800   IN      NS      ns-102-c.gandi.net.
cat-amania.com.         10800   IN      NS      ns-206-b.gandi.net.
```

We can then use dig to query using a specific nameserver:
```
dig @ns-206-b.gandi.net cat-amania.com
```

### TXT
get a list of text annotations for cat-amania.com:
```
dig cat-amania.com TXT +noall +answer
```
