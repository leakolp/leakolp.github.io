---
layout: post
title: factory reset your Drobo
subtitle: How to do a pinhole factory reset of the Drobo 5n NAS
tags: [homelab, drobo]
comments: false
---

I recently picked up a used Drobo 5n complete with disks. The password to it had long since been forgotten. To solve for this, I ended up doing a successful pinhole reset of 
the Drobo. Please see the following instructions [here](https://drobocommunity.m-ize.com/t/pinhole-reset/75489/28).

```bash
Ultimately i did this without drives in but i think it will also work with drives.

shut it down
unplug the power
Depress the pinhole switch (and from this point on keep this depressed till the end)
Hold down the power switch (and from this point on keep this depressed till the end) <-- this is the part i was missing out of there instructions
Regardless of pain, continue to hold the reset switch AND the power switch until you get the red LED for the top drive, OR (in theory) the drives go green.
After this i had the drobo showing up again in the dash board. When i added the old drives back in it seemed like it was trying to recover them, so i ended up doing another 
reset from the dashboard and that resolved the remaining problems.

Hope this helps future googlers
```

-Erin Kolp

