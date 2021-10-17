# misc/c-brother-1 (65 solves/476 points)

## Description:
AC01010 and JC01010's long lost twin has joined the [HSCTF discord](https://discord.gg/C9UMj3qN8a) and started a Youtube channel! Although they haven't uploaded any videos, they've made some customizations to some of their video watermarks.

Please respect the privacy of our organizers and do not OSINT any of our organizers. None of the flags are hidden on our own social media sites or other accounts. This challenge includes BC01010 and this person only. Do not visit any other profiles, including JC01010, AC, or any other organizers. Thank you.
## Solution:
I first went to the discord server and searched for '01010' and found the user BC01010. A [YouTube channel](https://www.youtube.com/channel/UCqZq81jZcdjAHQJ3UtAbdaA) was connected to the profile, but since there were no videos uploaded, I couldn't easily extract the watermark. After doing some research on channel watermarks, I found that this is the generic form for the watermark link: i.ytimg.com/an/{channelId}/featured_channel.jpg. The channel id can be found easily as it is the text after "UC" in https://www.youtube.com/channel/UCqZq81jZcdjAHQJ3UtAbdaA. After inserting the channel id in the generic watermark link, we see:
![image](https://i.ytimg.com/an/qZq81jZcdjAHQJ3UtAbdaA/featured_channel.jpg)

## Flag:
`flag{h1dd3n_wat3rm@rk}`