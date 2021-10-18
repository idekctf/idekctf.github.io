---
title: HSCTF - Misc
tags: misc
---

## misc/c-brother-1 (65 solves/476 points)

### Description:
AC01010 and JC01010's long lost twin has joined the [HSCTF discord](https://discord.gg/C9UMj3qN8a) and started a Youtube channel! Although they haven't uploaded any videos, they've made some customizations to some of their video watermarks.

Please respect the privacy of our organizers and do not OSINT any of our organizers. None of the flags are hidden on our own social media sites or other accounts. This challenge includes BC01010 and this person only. Do not visit any other profiles, including JC01010, AC, or any other organizers. Thank you.
### Solution:
I first went to the discord server and searched for '01010' and found the user BC01010. A [YouTube channel](https://www.youtube.com/channel/UCqZq81jZcdjAHQJ3UtAbdaA) was connected to the profile, but since there were no videos uploaded, I couldn't easily extract the watermark. After doing some research on channel watermarks, I found that this is the generic form for the watermark link: i.ytimg.com/an/{channelId}/featured_channel.jpg. The channel id can be found easily as it is the text after "UC" in https://www.youtube.com/channel/UCqZq81jZcdjAHQJ3UtAbdaA. After inserting the channel id in the generic watermark link, we see:
<img src="https://i.ytimg.com/an/qZq81jZcdjAHQJ3UtAbdaA/featured_channel.jpg" style="background-color:white;" />

### Flag:
flag{h1dd3n_wat3rm@rk}

## misc/c-brother-2 (49 solves/483 points) #

### Description: ##
It looks like AC01010 and JC01010's third twin has changed their profile picture!

Note: It may be helpful to solve c-brother-1 before c-brother-2

Please respect the privacy of our organizers and do not OSINT any of our organizers. None of the flags are hidden on our own social media sites or other accounts. This challenge includes BC01010 and this person only. Do not visit any other profiles, including JC01010, AC, or any other organizers. Thank you.

### Solution: ##

The descriptions hints at finding an old profile picture for the [YouTube channel](https://www.youtube.com/channel/UCqZq81jZcdjAHQJ3UtAbdaA) we found in c-brother-1. To do so, I used the Wayback Machine on all the pages on the channel. Eventually, I found this link: https://web.archive.org/web/20210614150352if_/https://www.youtube.com/channel/UCqZq81jZcdjAHQJ3UtAbdaA/about, where the profile picture seemed to be the flag.
![image](https://yt3.ggpht.com/Ug17FJY-SiCjr5JuAR_JIcBtmoUirZ2g6lVzm3NstW2K9gH5aH1jbuDaYF36-f4jdtTzd95f=s88-c-k-c0x00ffffff-no-rj)
It was a bit hard to read, so I right-clicked the profile picture to get: https://web.archive.org/web/20210614150354/https://yt3.ggpht.com/Ug17FJY-SiCjr5JuAR_JIcBtmoUirZ2g6lVzm3NstW2K9gH5aH1jbuDaYF36-f4jdtTzd95f=s88-c-k-c0x00ffffff-no-rj. 
I copied the portion of the link after "https://web.archive.org/web/20210614150354/" and changed "s88" to "s500" to see:
![image](https://yt3.ggpht.com/Ug17FJY-SiCjr5JuAR_JIcBtmoUirZ2g6lVzm3NstW2K9gH5aH1jbuDaYF36-f4jdtTzd95f=s500-c-k-c0x00ffffff-no-rj)

### Flag: ##
flag{f1ag_fl@g_fla6}

## misc/LsBlue

### Description: ##

Orca watching is an awesome pastime of mine!
Downloads: lsblue.png


### Solution: ##
From the category of this problem, we know that it is in the misc/forensic category. So, the first step I tried was to download the image and open it up in a hex editor to analyze the metadata. No flag. My next attempt was to run a stego tool on the image. Since the image is a png I went with my favorite tool: Zsteg. 

-Below is the command used in my shell: `zsteg -a lsblue.png`

The output gave me tons of crap to sift through but I'm lazy and didn't want to do that, so with a quick grep command for flag{. We were able to modify the command and get the flag.
```
New command: zsteg -a lsblue.png | grep "flag{"
Output: b1,b,lsb,xy         .. text: "flag{0rc45_4r3nt_6lu3_s1lly_4895131}"
```
And there it is!

### Flag: ##
flag{0rc45_4r3nt_6lu3_s1lly_4895131}

## misc/pallets-of-gold


### Description:
It doesn't really look like gold to me...
[pallets-of-gold.png]


### Solution:
Let's start by downloading the image and opening it up:
![image](/Users/SuperBeetleGamer/Desktop/pallets-of-gold.png)
Ok, not too much to work with. I then tried to run it through my favorite steg tool `zsteg` with `zsteg -a pallets-of-gold.png`
Yeah... Nothing
Then, I went to this website https://stegonline.georgeom.net/upload
Turing on LSB half and saving the downloaded image gives us this:
![image](/Users/SuperBeetleGamer/Downloads/pallets-of-gold.png)
WHICH IS THE FLAG!!!

### Flag: 
flag{plte_chunks_remind_me_of_gifs}

## misc/Return of the Intro to Netcat (662 solves/160 points)

### Description: ##
hey, netcat seems fun! (with a twist)
`nc return-of-the-intro-to-netcat.hsc.tf 1337`

### Solution: ##
After opening a terminal window and running `nc return-of-the-intro-to-netcat.hsc.tf 1337`, we see:
```
== proof-of-work: enabled ==
please solve a pow first
You can run the solver with:
    python3 <(curl -sSL https://goo.gle/kctf-pow) solve s.AACF.AAB49VrGq+T0PVB6R9bV92Rb
===================

Solution?
```
I opened another terminal window and ran `python3 <(curl -sSL https://goo.gle/kctf-pow) solve s.AACF.AAB49VrGq+T0PVB6R9bV92Rb`, which returned:
```
s.AAA5PRheLM93CpiPiRpLZwqciE7d19rTavx4VoIS1Oz6DRxHHFFghOCjFGIJlmbTfR5IWsTTM8rWz3mqtD02OM/sJ8QRxQT9u880J4MRfH462nqMIj9+igTbIg8K3nODylfy9z6rJIovsoMD7iDeydAsMTZu/tMQLzAkJgYvDOpV3SaksRgiZF9pDAUiVYQVTwdxrGQf8V/Iap4PUkY8FVQ+
```
I copied the result and gave it as input to the first terminal window, which returned: 
```
Correct
You got it! Here's what you're looking for: flag{the_cat_says_meow}
```

### Flag: ##
flag{the_cat_says_meow}