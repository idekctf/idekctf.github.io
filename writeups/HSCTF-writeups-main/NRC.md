# web/NRC (955 solves/107 points)

## Description: ##
Find the flag :)
> `no-right-click.hsc.tf`

## Solution:

As the link says, there's no right click on the website; however, with `Ctrl+Shift+C` we are able to see the developer console. We can navigate to the sources tab and then view `useless-files.css`:

```css
body {
    text-align: center;
    font-size: 5rem;
    font-family: 'Abril Fatface', cursive;
}
.small {
    margin-top: 50vh;
    font-size: 0.5rem;
}
/* cause i disabled it in index.js */
/* no right click = n.r.c. */
/* flag{keyboard_shortcuts_or_taskbar} */
```

And the the flag is on the last line.

## Flag:
flag{keyboard_shortcuts_or_taskbar}