# Report of Computer Architecture

* Stage:PA0
* Date: 2019.2.29

* All content in this stage are completed.

## Menu

[TOC]

## Questions

1. Why Windows is quite clumsy?

There are many aspects why Windows is far more complicated than linux. 

First of all is about fall back support. Handling capability to old applications in Windows is different from that in Linux. As development continues, fall-back support to programs should be considered. On Linux, many distributed version can be chosen for capability support and dependencies can be handled easily, while on Windows, it's implemented by stacking supportive code in one distribution (mainly for commercial purposes). Thus means old code or applications such as registration files (fall back supports & major source of bugs) and cmd.exe (substituded by PowerShell) cannot be removed from Windows, causing accumulation and eventually stacking up its complexity. 

Second is about usability. As known to all, Linux is designed for professional use while Windows is dedicated to common users, meaning auxiliary additions like GUI are implemented by default on Windows. Linux simpliy continues its way as command line interface (no worry for pros), reducing system loads and boosting efficiency.

2. Why 'poweroff' command requires superuser authentication?

Most of the time, Linux runs on a server, which cannot be poweroff on routines (otherwise it will paralyze any services it hosts). So preventing un-authorized users from shutting down a server is crucial for service stability.

3. Can't Memory allocation be larger?

First of all, xx-bits means how many bits a CPU can proceed in one tick. 

This limitation can be resolved by address indexing, however it will result in compromise of effiency and require multiple CPU ticks. For high velocity read-write devices like memory, it's not allowed.

For 32-bits system, memory is restricted to a maximum of 4GB. This is because of the limited length of address. In a computer, memory are located by an unique integer (preventing access conflicts). On 32-bits systems, such addresses are limited to 32 bits binary, means the maximum amount of memory is 2^32 bytes, equals to 4GB. Such restriction is eased on 64-bits systems, for integer used to locate memory exceed our daily needs (2^64 bytes = 16EB).

4. Linux distribution?

As mentioned above, there are variant distribution of Linux to choose from. Widely used distributions includes Debian, Fedora, Mandriva Linux, Red Hat, Ubuntu, CentOS, etc. Each distro are derived to serve different purposes. Few examples are enlisted below:

> * Debian, a non-commercial distribution and one of the earliest, maintained by a volunteer developer community with a strong commitment to free software principles and democratic project management

> * Ubuntu, a desktop and server distribution derived from Debian, maintained by British company Canonical Ltd.

> * CentOS, a distribution derived from the same sources used by Red Hat, maintained by a dedicated volunteer community of developers with both 100% Red Hat-compatible versions and an upgraded version that is not always 100% upstream compatible.

Most distros are GNU distros, which obeys guidelines from GNU. However, there're also non-GNU distributions, Android for example.

For general informations about Linux distros, refer to [Linux Distribution on Wikipedia](https://en.wikipedia.org/wiki/Linux_distribution)

5. A basic computer?

According to Von Neumann architecture, creating a machine running Hello World consists of the following parts:

   * a mass storage that stores the program as well as rendering info to the alphabet.
   * a memory that store info for high velocity.
   * a display device to show the words.
   * an input device to give commands.
   * ctrl flow that control the whole procedure, including renderer caching.

Procedure is: 
   1. read ctrl flow of hello world to the memory.
   2. according to the control flow, read string ("Hello world") from memory allocated to the application.
   3. call for rendering (Converting characters into bit mapping on screen).
   4. resources recycling.


## Content

**Note: PA0.6 is executed before PA0.3 to gain convenience.**

### PA0.1 Installing a GNU/Linux VM

> For language settings on current host OS, screenshots in this section will be in Japanese

#### Create new VM for Debian on `VMware Fusion`
* Open `Virtual Image Library`
    ![VM_Lib.jpg](./img1.jpg)
* Click "+", then select `Install from ISO image`
    ![VM_new.jpg](./img2.jpg)
* Config VM resources using default config
    ![VM_config.jpg](./img3.jpg)
    
#### Install Debian on VM

##### Boot VM, then follow installation procedure.
> 
Language: English
Region: US
Keyboard mapping: US
Host Name: Debian (made up)
Root Password: **Null** (for using sudo instead of root)
User Name: `qiujiaqi`
Password: **Confidential**
Time Zone: Pacific

##### Disk Partitioning

> For practicing, disk partitioning will be in manual mode.

There should be at least 2 partitions on the disk: a logical partition used as swap area, and a primary partition for file system. 

![VM_partition.jpg](./img7.jpg)

For selection on file sys, it's recommended to use journaling file system to enhance data security.
For advanced security, it's better to seperate crucial directory into different partition. It's usually used to prevent data flooding like DoS attacks. To config, modify mount point of each partition.
![VM_partition_advanved.jpg](./img8.jpg)

##### Start Online Installation

Choose nearest mirror: mirror.163.com
Then click install to continue.

##### GRUB config
>
GNU GRUB (short for GNU GRand Unified Bootloader, commonly referred to as GRUB) is a boot loader package from the GNU Project. GRUB is the reference implementation of the Free Software Foundation's Multiboot Specification, which provides a user the choice to boot one of multiple operating systems installed on a computer or select a specific kernel configuration available on a particular operating system's partitions.

Click yes, then select /dev/sha (OS mount point)

##### Done Installing Debian

![VM_done.jpg](./img10.jpg)

### PA0.2 First Step into Linux

#### Log in to Debian

#### Check Disk Consumption

run command 
    
```bash
df -h
```

![VM_diskConsumption](./img14.jpg)

#### Power off
Run command
```bash
sudo poweroff
```
Enter Passcode, then continue.

### PA0.3 Installing Basic Tools

 For online installation, `/etc/apt/sources.list` will be well configured, thus it's not necessary to install tools from images. The following steps are for oral demonstration and skipped.

#### Mount image to Debian

Insert image to VM

Then execute command to mount the image
    
```bash
apt-cdrom add
```

Note that after installation from image, make sure to remove relavant source from `/etc/apt.sources.list`, otherwise it will trigger "hardware changed" error.

#### Install basic tools to Debian

Since `SSH` and `sudo` are configed in Debian installation, the only tools to install is `vim`.
Execute command:

```bash
sudo apt-get install vim
```

### PA0.4 Installing Additional Tools

#### Config Network 

For Online installation, configuring network adaptor via `/etc/network/interface` can be skipped.

#### Config Source

As mentioned above, failing to update image source in `/etc/apt.sources.list` will result in failure.

```bash
sudo vim /etc/apt.sources.list
```
For network enviornment, sources other than provided are added.

After editing, execute command to update:

```bash
sudo apt-get update
```

#### More tools for PA

>
Packages can be stack up to stop being annoyed by entering passcode on each installation, especially while handling massive amount of packages.

Execute following commands to install all tools for PA:

```bash
sudo apt-get install man build-essential gcc-doc gdb git gcc-multilib libreadline-dev libsdl2-dev qemu-system-x86
```

If all tools are installed, redoing command above will check update for each package.

![VM_toolInstalled](./img16.jpg)

### PA0.5 More Exploration

#### Optimize usability of `vim`

Optimization of vim depends on its internal settings, which can be turned on temporary using `:` commands in the application. However, such settings will be wiped each time vim exits. To avoid troubles, vim refers to `~/.vimrc` under user directory to automatically fill in these commands.

Here is a demo of `.vimrc` config

![VM_vimrc](./img17.jpg)

#### Compile and Run Hello World

```bash
vim hello.c
vim Makefile
make
./hello
```
![VM_helloWorld](./img20.jpg)

### PA0.6 Logging in and Transfering Files via `SSH`

#### Seek for IP Address on Guest OS

Run:

```bash
ip addr
```

Refer to ip address under virtual network adaptor.

#### Log in via `SSH`

Run on Host OS: (ip address are dynamic due to network config, `192.168.231.137` in this case)

```bash
ssh qiujiaqi@192.168.231.137
```

#### Transfering Files between different hosts

We can use the following command to transfer files between hosts
```bash
tar cj file | ssh username@ip_addr 'tar xvjf -'
ssh username@ip_addr 'tar cj file' | tar xvjf -
```
We can also use `scp` (based on `SSH`) for files trransfering

```bash
scp [src] [tar]
```

TEST: Transfering `a.out` under `qiujiaqi@192.168.231.137:~/` dir to `~/Developer`

```bash
scp qiujiaqi@192.168.231.137:~/a.out ~/Developer/
```

![VM_fileTransfer](./img18.jpg)

#### Config X11 Service

Install `XQuarz` on HostOS, then install `xclock` on Debian

```bash
sudo apt-get install x11-apps
```

On XQuarz, connect to GuestOS via `SSH` with `x11forwarding` enabled.

```bash
ssh -X qiujiaqi@192.168.231.137 
```

Test run xclock:

```bash
xclock
```

![VM_xclock](./img19.jpg)

#### Config Time
> In VMware, time and date are automatically synced with GuestOS. So it can be skipped.

![VM_timeSync](./img21.jpg)

#### Backup VM

VMware Fusion implemented a feature "Snapshot" to backup VM to certain condition.

![VM_screenshot](./img22.jpg)

### PA0.7 Acquiring Source Code for PAs

Acuire PA files using:

```bash
git clone https://github.com/jinhang1997/ics2017 ics2017
```

Create new branch:

```bash
git checkout pa0
```
Make modification:

```bash
vim ./nemu/Makefile.git
git status
git diff
```

>
qiujiaqi@debian:~/ics2017$ git status
On branch pa0
Changes not staged for commit:
(use "git add <file>..." to update what will be committed)
(use "git checkout -- <file>..." to discard changes in working directory)
modified:   nemu/Makefile.git
no changes added to commit (use "git add" and/or "git commit -a")
qiujiaqi@debian:~/ics2017$ 

>
diff --git a/nemu/Makefile.git b/nemu/Makefile.git
index 7b005db..067edf0 100644
--- a/nemu/Makefile.git
+++ b/nemu/Makefile.git
@@ -1,4 +1,4 @@
-STU_ID = 161220000
+STU_ID = 161730320

Commit changes:
```bash
git add .
git commit
```
>
[pa0 3ae60bb] First Commit
1 file changed, 1 insertion(+), 1 deletion(-)

Push changes to Github:

```bash
git remote add myrepo https://github.com/[blurred for privacy]/ics2017.git
git push -u myrepo master
git push -u myrepo pa0
```
>
Username for 'https://github.com': **[Censored]**
Password for 'https://**[Censored]**@github.com': 
Counting objects: 1019, done.
Compressing objects: 100% (857/857), done.
Writing objects: 100% (1019/1019), 2.21 MiB | 744.00 KiB/s, done.
Total 1019 (delta 120), reused 1019 (delta 120)
remote: Resolving deltas: 100% (120/120), done.
To https://github.com/**[Censored]**/ics2017.git
* [new branch]      master -> master
Branch master set up to track remote branch master from myrepo.

## Troubles Encountered

Q. x11 not supported in terminal.app?

A. terminal.app is designed for providing text-base running environment, which does not support graphical components like x11 contents. XQuarz is recommended to enable x11.

Q. connection back to HostOS unsuccessful?

A. this is because firewall on macOS is off by default. To enable it, refer to `settings > sharing`. Not recommended for security.

Q. Why I didn't config a dual start?

A. The dicision I made not to perform dual start depends on 2 things: routine requirements and machine lifespan. As routine requirement, running a 32-bit system does not meet daily requirement. Most software we used today are 64-bits, which is not capable to run on 32-bit platform, as well as many manufactorers start dropping support of 32-bits programs on both hardware and software platform (Apple for instance), meaning 32-bits platforms are fading out. Also, lack of backup function means more complexity on data security and less chance for recovery on malfunctions. Another aspect, the machine lifespan. Due to capability support, running a 32-bit platform on 64-bit hardware causes extra burden to CPU and also results in system unstability and risks losing data. Nevertheless, without virtualization, partitioning a disk with different non-optimized file system could cause great damage to the disk, not mentioning risking ruining the swap area crucial to system bootup and disk decryption [my giga bytes of *** contents].

## Conclusion

This is the end of PA0. Since I've been working with UNIX for periods of time (including server setup, system config), it's almost a routine for me. However, gaining additional knowledge such as git is still beneficial to future development (To be honest, I've never used git before). And of course, reading the documents enhances my English reading skills and provides me a good chance to implement English into practice.

## Additions

A principle as an administrator:

>We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:
1) Respect the privacy of others.
2) Think before you type.
3) With great power comes great responsibility.

and a rule:

> **K**eep **I**t **S**imple and **S**tupid.
