---
layout: post
title: BTLO PhishyV1 Walkthrough
---
![Alt](https://bohansec.com/assets/PhishyV1/cover.png "Security Blue Team")

The writeups will be a series to document how I solved each scenario on BTLO (Blue Team Labs Online), hope you will enjoy it :)

## PhishyV1 Video Walkthough 

<div class="youtube-wrapper">
    <iframe 
            src="https://www.youtube.com/embed/A2kwlPcv7Xc"
            frameborder="0"
            allow="autoplay; encrypted-media"
            allowfullscreen></iframe>
</div>



## Scenario

***You have been sent a phishing link - It is your task to investigate this website and find out everything you can about the site, the actor responsible, and perform threat intelligence work on the operator(s) of the phishing site.*** 

***Warning: The website and kit you see is the lab is REAL. Exercise caution when interacting with the malicious website and do not enter any sensitive information***

## Tools
Web Browser 

Text Editor 

Linux CLI 

## Difficulty
- Easy

## Reading Material
N/A

## Scenario Questions

### The HTML page used on securedocument.net is a decoy. Where was this webpage mirrored from, and what tool was used? (Use the first part of the tool name only) (4 points)

Look for the webroot for the phishy website, then open the HTML source page to look for the HTML comment.

![screenshot](https://bohansec.com/assets/PhishyV1/1.PNG "screenshot")

Answer:

{% highlight js %}
61.221.12.26/cgi-sys/defaultwebpage.cgi, HTTrack
{% endhighlight %}

### What is the full URL of the background image which is on the phishing landing page?

On the landing page, right-click, and follow the background image link.

![screenshot](https://bohansec.com/assets/PhishyV1/2.PNG "screenshot")

Answer:

{% highlight js %}
http://Securedocument.net/secure/L0GIN/protected/login/portal/axCBhIt.png
{% endhighlight %}

### What is the name of the php page which will process the stolen credentials?

Lookup the HTML source code, we see the jeff.php is used to process the credentials.

![screenshot](https://bohansec.com/assets/PhishyV1/3.PNG "screenshot")

Answer:

jeff.php

### What is the SHA256 of the phishing kit in ZIP format? (Provide the last 6 characters)

We will find the zip file, download it, and sha256sum on it.

![screenshot](https://bohansec.com/assets/PhishyV1/4.PNG "screenshot")
![screenshot](https://bohansec.com/assets/PhishyV1/5.PNG "screenshot")

Answer:

fa5b48

### What email address is setup to receive the phishing credential logs?

Look for the jeff.php source code.

![screenshot](https://bohansec.com/assets/PhishyV1/6.PNG "screenshot")

Answer:

boris.smets@tfl-uk.co

### What is the function called to produce the PHP variable which appears in the index1.html URL? 

The timestamp is added after the URL, we can look for the index.html file in the unzipped file. There is a function used for producing the timestamp in PHP.

![screenshot](https://bohansec.com/assets/PhishyV1/7.PNG "screenshot")

Answer:

getTime()

### What is the domain of the website which should appear once credentials are entered?

Look for the redirecting Javascript code for the landing page.

![screenshot](https://bohansec.com/assets/PhishyV1/8.PNG "screenshot")

Answer:

Office.com

### There is an error in this phishing kit. What variable name is wrong causing the phishing site to break? (Enter any of 4 potential answers) 

From the HTML source code, we see the variable are not the same compares to the PHP code, so we know there are errors with the PHP code. The variable should be the same as the HTML code.

![screenshot](https://bohansec.com/assets/PhishyV1/9.PNG "screenshot")
![screenshot](https://bohansec.com/assets/PhishyV1/10.PNG "screenshot")

Answer:

User1

That's all for today, hope you enjoyed the walkthrough and learned something new! I will keep posting more BTLO writeups as I make progress on the platform. Thank you for stopping by :) 




