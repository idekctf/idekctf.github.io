# misc/LsBlue

## Description: ##

Orca watching is an awesome pastime of mine!
Downloads: lsblue.png


## Solution: ##
From the category of this problem, we know that it is in the misc/forensic category. So, the first step I tried was to download the image and open it up in a hex editor to analyze the metadata. No flag. My next attempt was to run a stego tool on the image. Since the image is a png I went with my favorite tool: Zsteg. 

-Below is the command used in my shell: `zsteg -a lsblue.png`

The output gave me tons of crap to sift through but I'm lazy and didn't want to do that, so with a quick grep command for flag{. We were able to modify the command and get the flag.
```
New command: zsteg -a lsblue.png | grep "flag{"
Output: b1,b,lsb,xy         .. text: "flag{0rc45_4r3nt_6lu3_s1lly_4895131}"
```
And there it is!

## Flag: ##
flag{0rc45_4r3nt_6lu3_s1lly_4895131}