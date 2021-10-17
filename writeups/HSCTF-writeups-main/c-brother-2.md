# misc/c-brother-2 (49 solves/483 points) #

## Description: ##
It looks like AC01010 and JC01010's third twin has changed their profile picture!

Note: It may be helpful to solve c-brother-1 before c-brother-2

Please respect the privacy of our organizers and do not OSINT any of our organizers. None of the flags are hidden on our own social media sites or other accounts. This challenge includes BC01010 and this person only. Do not visit any other profiles, including JC01010, AC, or any other organizers. Thank you.

## Solution: ##

The descriptions hints at finding an old profile picture for the [YouTube channel](https://www.youtube.com/channel/UCqZq81jZcdjAHQJ3UtAbdaA) we found in c-brother-1. To do so, I used the Wayback Machine on all the pages on the channel. Eventually, I found this link: https://web.archive.org/web/20210614150352if_/https://www.youtube.com/channel/UCqZq81jZcdjAHQJ3UtAbdaA/about, where the profile picture seemed to be the flag.
![image](https://yt3.ggpht.com/Ug17FJY-SiCjr5JuAR_JIcBtmoUirZ2g6lVzm3NstW2K9gH5aH1jbuDaYF36-f4jdtTzd95f=s88-c-k-c0x00ffffff-no-rj)
It was a bit hard to read, so I right-clicked the profile picture to get: https://web.archive.org/web/20210614150354/https://yt3.ggpht.com/Ug17FJY-SiCjr5JuAR_JIcBtmoUirZ2g6lVzm3NstW2K9gH5aH1jbuDaYF36-f4jdtTzd95f=s88-c-k-c0x00ffffff-no-rj. 
I copied the portion of the link after "https://web.archive.org/web/20210614150354/" and changed "s88" to "s500" to see:
![image](https://yt3.ggpht.com/Ug17FJY-SiCjr5JuAR_JIcBtmoUirZ2g6lVzm3NstW2K9gH5aH1jbuDaYF36-f4jdtTzd95f=s500-c-k-c0x00ffffff-no-rj)

## Flag: ##
flag{f1ag_fl@g_fla6}