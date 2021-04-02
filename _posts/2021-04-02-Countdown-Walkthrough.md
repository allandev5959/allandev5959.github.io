---
layout: post
title: BTLO Countdown Walkthrough
---
![Alt](https://bohansec.com/assets/Countdown/coverpage.jpg "Security Blue Team")

## Countdown Video Walkthough 

<iframe width="560"
        height="315"
        src="https://youtu.be/YrfyzyiXvek"
        frameborder="0"
        allow="autoplay; encrypted-media"
        allowfullscreen></iframe>

## Scenario

***NYC Police received information that a gang of attackers has entered the city and are planning to detonate an explosive device. Law enforcement have begun investigating all leads to determine whether this is true or a hoax.***

***Persons of interest were taken into custody, and one additional suspect named ‘Zerry’ was detained while officers raided his house. During the search they found one laptop, collected the digital evidence, and sent it to NYC digital forensics division.***

***Police believe Zerry is directly associated with the gang and are analyzing his device to uncover any information about the potential attack.***

***Disclaimer: The story, all names, characters, and incidents portrayed in this challenge are fictitious and any relevance to real-world events is completely coincidental.***

## Tools
- Autopsy 
- Window File Analyzer 
- WinPrefetchView 
- Jumplist Explorer 
- SQLite DB Browser 

## Difficulty
Medium

## Scenario Questions

### Verify the Disk Image. Submit SectorCount and MD5

The first question is quite stright forward, we can find the answer under "Desktop -> Investigation Files-> Disk Image -> Zerry -> Zerry.E01(text file)", we can see the md5 presented in the file.

![screenshot](https://bohansec.com/assets/Countdown/1.PNG "screenshot")

Answer: 
5c4e94315039f890e839d6992aeb6c58

### What is the decryption key of the online messenger app used by Zerry?

The online messenger key could locate in the file system. Open the "Countdown.aut" file under "Desktop -> Countdown" in Autopsy.

![screenshot](https://bohansec.com/assets/Countdown/2.PNG "screenshot")

Under the "Zerry.E01 disk -> Volumn 3 -> Users -> ZerryD -> App Data -> Roaming", we find the secure messenger used by the criminals is Signal. We can extract the whole folder to the "Export" for futhur investigation.

![screenshot](https://bohansec.com/assets/Countdown/3.PNG "screenshot")

For decryption key, let's checkout the app's config file to see what we can find. We find the raw key in the "config.json" file! 

![screenshot](https://bohansec.com/assets/Countdown/3-1.PNG "screenshot")


Answer:
{% highlight js %}
c2a0e8d6f0853449cfcf4b75176c277535b3677de1bb59186b32f0dc6ed69998
{% endhighlight %}

### What is the registered phone number and profile name of Zerry in the messenger application used?

With the decryption key in hand, we can use it to decrypt the database and read the data from it. We will open the Signal Database via SQLiteBrowser under tools folder. Use the raw key we found early to decrypt the database. Notice I chose the raw key and added "0x" before my key.

{% highlight js %}
0xc2a0e8d6f0853449cfcf4b75176c277535b3677de1bb59186b32f0dc6ed69998
{% endhighlight %}

Now, we have a decrypted database.

![screenshot](https://bohansec.com/assets/Countdown/4.PNG "screenshot")
![screenshot](https://bohansec.com/assets/Countdown/5.PNG "screenshot")
![screenshot](https://bohansec.com/assets/Countdown/6.PNG "screenshot")

Under Conversation table, we can find the Zerry's username and phone number. Note the profile name has an emoji "fire" in it if you search on google.

![screenshot](https://bohansec.com/assets/Countdown/7.PNG "screenshot")
![screenshot](https://bohansec.com/assets/Countdown/8.PNG "screenshot")

Answer:
+13026482364, ZerryThe🔥 

### What is the email id found in the chat? 

We can find the email used under "messages_fts" table.

![screenshot](https://bohansec.com/assets/Countdown/9.PNG "screenshot")

Answer:
eekurk@baybabes.com

### What is the filename(including extension) that is received as an attachment via email?

From the conversation, we find the criminals trying to bomb the city. Zerry wants to know the bomb time. So, a image file contains date and time was being downloaded. We can locate the image file downloaded at Autopsy Recent Documents. The emoji for the filename are "time" and "Calender".

![screenshot](https://bohansec.com/assets/Countdown/10.PNG "screenshot")
![screenshot](https://bohansec.com/assets/Countdown/11.PNG "screenshot")

Answer:
⌛📅.PNG

### What is the Date and Time of the planned attack?

***Hint: It is possible this information was stored in a Sticky Note. Find this in /Users/Zerry/AppData/Local/Packages and export it. Open the plum.sqlite database file in the /LocalState/ directory using SQLiteDatabaseBrowser. Find which table the notes are stored in. We can work out how to decrypt the encoded result by checking recent sites visited via Tor in Autopsy by looking at /Tor Browser/Browser/TorBrowser/Data/Browser/proile.default/places.sqlite.***

I used hint provided by BTLO for this question, we need to extract the "thumbcache256.db" under "/Users/Zerry/AppData/Local/Microsoft/Windows/Explorer", then use "Thumbcache Viewer" under tools folder to open the database to view the images.

![screenshot](https://bohansec.com/assets/Countdown/12.PNG "screenshot")
![screenshot](https://bohansec.com/assets/Countdown/13.PNG "screenshot")

Answer:
01-02-2021 9:00 am

### What is the GPS location of the blast? The format is the same as found in the evidence . [Hint: Encode(XX Degrees,XX Minutes, XX Seconds)] 

***Hint: It is possible this information was stored in a Sticky Note. Find this in /Users/Zerry/AppData/Local/Packages and export it. Open the plum.sqlite database file in the /LocalState/ directory using SQLiteDatabaseBrowser. Find which table the notes are stored in. We can work out how to decrypt the encoded result by checking recent sites visited via Tor in Autopsy by looking at /Tor Browser/Browser/TorBrowser/Data/Browser/proile.default/places.sqlite.***

The last question is quite tricky since you need to be very familar with the file system in order to solve it. I used hint provided by BTLO for this question. First, we need to locate the stickynote the criminal left based on the hint. Frist, lets export the folder the file located at from Autopsy. The folder is "/Users/Zerry/AppData/Local/Packages"

![screenshot](https://bohansec.com/assets/Countdown/14.PNG "screenshot")

By searching the file "plum.sqlite" in the exported folder, we can locate its location and use SQLiteBrowser to open it.

![screenshot](https://bohansec.com/assets/Countdown/15.PNG "screenshot")
![screenshot](https://bohansec.com/assets/Countdown/16.PNG "screenshot")

Locate the note under "Note" table. We got the encoded GPS location. Now, we need to find a way to decode it!

Encoded GPS Location:
{% highlight js %}
\id=f92c091d-7161-4cce-8deb-b53438d8238c 40 qrterrf 45 zvahgrf 28.6776 frpbaqf A, 73 qrterrf 59 zvahgrf 7.944 frpbaqf J
{% endhighlight %}

![screenshot](https://bohansec.com/assets/Countdown/17.PNG "screenshot")

Based on the hint, we knew they used Tor browser, so lets find the Tor browser folder and extract the file "places.sqlite".

File full path:
{% highlight js %}
/Tor Browser/Browser/TorBrowser/Data/Browser/proile.default/places.sqlite
{% endhighlight %}

![screenshot](https://bohansec.com/assets/Countdown/18.PNG "screenshot")

Open the "places.sqlite" with the SqliteBrowser, under table "moz_places" we find the encoded website is "https://rot13.com/". 

![screenshot](https://bohansec.com/assets/Countdown/19.PNG "screenshot")

We can perform the decode with the same website.

![screenshot](https://bohansec.com/assets/Countdown/20.PNG "screenshot")

Answer:
40 degrees 45 minutes 28.6776 seconds N, 73 degrees 59 minutes 7.944 seconds W


That's all for today, hope you enjoyed the walkthrough and learned something new! I will keep posting more BTLO writeups as I make progress on the platform. Thank you for stoping by :) 