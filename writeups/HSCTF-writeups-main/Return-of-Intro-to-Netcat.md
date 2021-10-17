# misc/Return of the Intro to Netcat (662 solves/160 points)

## Description: ##
hey, netcat seems fun! (with a twist)
`nc return-of-the-intro-to-netcat.hsc.tf 1337`

## Solution: ##
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

## Flag: ##
flag{the_cat_says_meow}