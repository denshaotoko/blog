---
title: Toolless Display Repair
date: 2025-09-06
author: denshaotoko
categories:
---

# TL;DR
As it turns out, it is possible to get from a non-working timer to a working one just by holding the minus button on the touchscreen for a long period of time.

# Toolless Repair?
In May user tjoaNe? on + [kaffee-netz.de](https://www.kaffee-netz.de/) shared his finding in this [post](https://www.kaffee-netz.de/threads/infos-eureka-mignon-display-timer-problem.163630/post-2466457).
The gist of it is what you can read in the TL;DR: If you run into the problem, just press the minus button and decrease the timer setting until you have a working timer.
How long does this take? Good question, let's have a look.

## Setup
I started with my vanilla hex file I which I read from a spare part last year and changed the value for the single dose timer to the maximum value.
Since the timer value is stored in two bytes, the maximum value for it is ```FFFF``` or 65535 in decimal (meaning 65535 tenths of a second).
This is what the "interesting" 24 bytes from the EEPROM look like on a brand new part:
```
:10 E000 00 3C0000006E0000000100000000000000 65
:10 E010 00 00000000000000000000010000000000 FF
:10 E020 00 00000000000000000000000000000000 F0
```
And this is what it looks like when the single dose timer is set to ```FFFF```:
```
:10 E000 00 FF00FF006E0000000100000000000000 A3
:10 E010 00 00000000000000000000010000000000 FF
:10 E020 00 00000000000000000000000000000000 F0
```
After the edit, I put the display back into my machine and when I swithced the grinder on, the screen displayed 3,5 seconds for the single doese timer.
The explanatione here is probably that only the last two digits are shown on the display.
Hex ```FFFF``` is 65535 in decimal and the display only shows the 35 as 3,5 seconds.

Great, now let's find out how long the it takes to get the timer working again.


## Testing
Starting at ```FFFF``` represents the worst case scenario.
So how long do I need to keep the minus button pressed to go from ```FFFF``` to 0,2 seconds (the lowest setting for the timer)?
For this, I measured the actual time necessary to lower the timer's setting by 10 seconds.
I repeated this a couple of times and it was always inbetween 2 and 3 seconds.
At worst, going from 6553,5 seconds to 0,2 seconds should take somewhere between 22 and 33 minutes.

With this estimate, I reflashed the display to set the single dose timer to  ```FFFF``` again and then started the test run.
I switched the grinder on and started pressing the minus button.
After holding the button for around 23 minutes, the timer arrived at 0,2 seconds and I could not decrease the value any further.
Once I reached 0,2 secondes, I could freely adjust the timer again within the regular interval of 0,2 to 30 secondes.

On the left: After manually setting the max value.
On the right After holding the minus button for around 23 minutes.

![single dose timer before]({{ site.baseurl }}/assets/images/timer-1-max-value-before.jpg){: width="49%"}
![single dose timer after]({{ site.baseurl }}/assets/images/timer-1-max-value-after.jpg){: width="49%"}
![screenshot stopwatch]({{ site.baseurl }}/assets/images/screenshot-stopwatch.png){: width="99%"}

I also read out the EEPROM to check if anything else had changed besides the values for the single dose timer.
Here is what the EEPROM looks like after arriving at 0,2 seconds:
```
:10 E000 00 020000006E0000000100000000000000 9F
:10 E010 00 00000000000000000000010000000000 FF
:10 E020 00 00000000000000000000000000000000 F0
```
As it turns out: nothing changed besides the area for the single dose timer is now showing ```02``` for 0,2 seconds, great.

## Summary
It is possible to repair the timer problem that is affecting the display units in [certain Eureka Mignon models]({{ site.baseurl }}{% post_url 2024-10-31-eureka-touchscreen-repair%}#other-eureka-grinders) without using any tools at all.
This is great news!
It takes a bit more than 20 minutes in the worst case scenario but that is the only downside.

It is still unclear what is actually causing the problem but at least there is a way to get a working grinder agaian without any disassamly or flashing microcontrollers.