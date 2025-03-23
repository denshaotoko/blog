---
title: Eureka Mignon EEPROM Content
date: 2025-03-21
author: denshaotoko
categories:
---

# What about the EEPROM?
In [my post]({{ site.baseurl }}{% post_url 2024-10-31-eureka-touchscreen-repair%}) regarding the display issue, I wrote how understanding the EEPROM contents might help to better understand the underlying issue.
The only differences between a working and a non-working display are the values stored in the EEPROM of the microcontroller.

I finally had some time at my hands and decided to try and find out what is stored where in the EEPROM of our beloved Eureka Mignon grinders.
Without any documentation I only came up with the following process to find out what bytes of the EEPROM area represent what setting of the grinder:
1. Flash a vanilla hex file onto a display.
2. Put that "new" display into the grinder.
3. Change one setting e.g., the single dose timer setting or the contrast value.
4. Read out the microcontroller and store the hex file.
5. Compare the vanilla hex file and the new one that contains one changed setting.
6. Rinse and repeat.

# Mapping the EEPROM
One key finding during my initial analysis was that a working display differs from a non-working one only in the first 24 bytes of the EEPROM.
While the whole EEPROM of a PIC16F1936 is 256 bytes in size, only the first 24 bytes are apparently used to store information.

As outline above, I started with flashing a display so that it is like a new part from the factory - at least software-wise.
From there I changed some tiny setting and read out the microcontroller after the change.
I repeated this process many times and always changed different settings on the touchscreen.
After comparing all the changes in the hex file collection, here is the most likely layout of the relevant 24 bytes of the EEPROM.
Note: I also updated the [table]({{ site.baseurl }}{% post_url 2024-10-31-eureka-touchscreen-repair%}#eeprom-contents) in the previous post

| Byte | Value | Comment                                         |
|------|-------|-------------------------------------------------|
| 00   | 3C    | Single dose timer lower byte                    |
| 01   | 00    | Single dose timer upper byte                    |
| 02   | 6E    | Double dose timer lower byte                    |
| 03   | 00    | Double dose timer upper byte                    |
| 04   | 01    | Single dose counter 1st byte (lower)            |
| 05   | 00    | Single dose counter 2nd byte                    |
| 06   | 00    | Single dose counter 3rd byte                    |
| 07   | 00    | Unknown                                         |
| 08   | 00    | Double dose counter 1st byte (lower)            |
| 09   | 00    | Double dose counter 2nd byte                    |
| 10   | 00    | Double dose counter 3rd byte                    |
| 11   | 00    | Unknown                                         |
| 12   | 00    | Contrast setting                                |
| 13   | 01    | Last selected timer/mode (01 single, 02 double) |
| 14   | 00    | Continuous mode (00 disabled, 01 enabled)       |
| 15   | 00    | Timer lock status (00 unlocked, 01 locked)      |
| 16   | 00    | Unknown                                         |
| 17   | 00    | Unknown                                         |
| 18   | 00    | Unknown                                         |
| 19   | 00    | Unknown                                         |
| 20   | 00    | Unknown                                         |
| 21   | 00    | Unknown                                         |
| 22   | 00    | Unknown                                         |
| 23   | 00    | Unknown                                         |

Despite all the changes I made to the grinder's settings, the values in the 10 bytes marked "unknown" did not change.
While I found out what the other 14 bytes are used for (most likely), I have no idea if the other 10 bytes are used for anything.

# EEPROM Samples
With a better understanding of what is stored where in the EEPROM I decided to have a look at the two hex files I had read from my two touchscreens when they exhibited the problem: ```PIC16F1936-06022020-Faulty.hex``` and ```PIC16F1936-03052021-Faulty.hex```.
Below are the relevant parts from the EEPROM of these two hex files and from the vanilla hex file.
I extracted the three lines representing the first 24 bytes of the EEPROM from each of the three hex files for brevity.

**Vanilla EEPROM**
```
1028 :10E000003C0000006E000000010000000000000065
1029 :10E0100000000000000000000000010000000000FF
1030 :10E0200000000000000000000000000000000000F0
```

**Faulty EEPROM #1 ```PIC16F1936-06022020-Faulty.hex```**
```
1028 :10E00000E500FC00400000009C0000000100000052
1029 :10E010006B0000000000000006000200010000008C
1030 :10E02000400012000000000000000000000000009E
```

**Faulty EEPROM #2 (```PIC16F1936-03052021-Faulty.hex```)**
```
1028 :10E00000AF00FE003A00000099000C000000000084
1029 :10E010001000FF0000000000F600020001000000F8
1030 :10E02000C9000D000000000000000000000000001A
```

Without the data from the hex record format and padding, the 24 bytes for the three samples looks as follows.
Details on the hex record fields and how the padding works can be found [here]({{ site.baseurl }}{% post_url 2024-10-31-eureka-touchscreen-repair%}#analysis). 

**Vanilla EEPROM**
```
3C 00 6E 00 01 00 00 00
00 00 00 00 00 01 00 00
00 00 00 00 00 00 00 00
```

**Faulty EEPROM #1  ```PIC16F1936-06022020-Faulty.hex```**
```
E5 FC 40 00 9C 00 01 00
6B 00 00 00 06 02 01 00
40 12 00 00 00 00 00 00
```

**Faulty EEPROM #2 (```PIC16F1936-03052021-Faulty.hex```)**
```
AF FE 3A 00 99 0C 00 00
10 FF 00 00 F6 02 01 00
C9 0D 00 00 00 00 00 00
```

# EEPROM Differences
After getting the relevant information I wrote everything into a table and looked if I could find values that looked suspicous, weired, or somehow wrong.
Here is the table I used to compare the values and find the "anomalies".

| Byte | Vanilla | Faulty #1  | Faulty #2 | Comment                                         |
|------|---------|------------|-----------|-------------------------------------------------|
| 00   | 3C      |  E5        | AF        | Single dose timer lower byte                    |
| 01   | 00      |  FC        | FE        | Single dose timer upper byte                    |
| 02   | 6E      |  40        | 3A        | Double dose timer lower byte                    |
| 03   | 00      |  00        | 00        | Double dose timer upper byte                    |
| 04   | 01      |  9C        | 99        | Single dose counter 1st byte (lower)            |
| 05   | 00      |  00        | 0C        | Single dose counter 2nd byte                    |
| 06   | 00      |  01        | 00        | Single dose counter 3rd byte                    |
| 07   | 00      |  00        | 00        | Unknown                                         |
| 08   | 00      |  6B        | 10        | Double dose counter 1st byte (lower)            |
| 09   | 00      |  00        | FF        | Double dose counter 2nd byte                    |
| 10   | 00      |  00        | 00        | Double dose counter 3rd byte                    |
| 11   | 00      |  00        | 00        | Unknown                                         |
| 12   | 00      |  06        | F6        | Contrast setting                                |
| 13   | 01      |  02        | 02        | Last selected timer/mode (01 single, 02 double) |
| 14   | 00      |  01        | 01        | Continuous mode (00 disabled, 01 enabled)       |
| 15   | 00      |  00        | 00        | Timer lock status(00 unlocked, 01 locked)       |
| 16   | 00      |  40        | C9        | Unknown                                         |
| 17   | 00      |  12        | 0D        | Unknown                                         |
| 18   | 00      |  00        | 00        | Unknown                                         |
| 19   | 00      |  00        | 00        | Unknown                                         |
| 20   | 00      |  00        | 00        | Unknown                                         |
| 21   | 00      |  00        | 00        | Unknown                                         |
| 22   | 00      |  00        | 00        | Unknown                                         |
| 23   | 00      |  00        | 00        | Unknown                                         |

When looking at the values in the table above, a few things stick out:
1. The single dose timer values look suspiciously high in both cases.
2. The dose count values appear high (single dose counter "Faulty #1", double dose counter "Faulty #2").
3. Weid contrast setting for "Faulty #2"
4. Some of the "unknown" fields contain data for the "faulty" hex files.

## High Single Dose Timer Values
I checked the maximum values for the single and the double dose timer and they are 30,0 seconds and 50,0 seconds, respectively.
For the single dose timer, the max value of 30,0 seconds would be stored as a hex representation of 300 tenth of a second.
In the same way, the max value of 50,0 seconds for the double dose timer would be stored as the hex representation of 500 tenth of a second.
At most, the single dose timer would be ```12C``` and the double dose timer ```1F4```.
For "Faulty #1", the single dose timer is set to ```FCE5``` and this value exceeds ```12C```by far.
This becomes even more apparent when looking at the decimal representation of  ```FCE5```, 64.741.
With this value, the timer would run for 6.474 seconds or 107 minutes.
Things look very much the same for the values from "Faulty #2".
The single dose timer is set to ```FEAF``` which is 65.199 in decimal - representing about 108 minutes.

## High Dose Count Values
Immediately noticeable are two very high numbers in the EEPROM area where the dose counts are being stored.
For "Faulty #1" there is ```1009C``` for the single dose count and for "Faulty #2" the value ```FF10``` is stored for the double dose count.
```1009C``` in decimal representation is 65.692 and ```FF10``` is 65.296.
To get to these numbers with normal use of the grinder, it would be necessary to grind coffee four times a day over a period of forty years.
These high values are very suspicious not only given the fact that I got this data from my two faulty displays which were in use for ca. one year and ca. two years before they went bad.

## Contrast Setting
"Faulty #2" has a weird looking contrast setting of ```F6``` which is 246 in decimal.
The grinder supports contrast settings of 0 to 7.
A value of 256 is way out of the configurable interval and hence is verly likely to be a corrupted value.

## Data in Bytes 16/17
Both faulty displays have data in bytes 16 and 17 that is neither in the vanilla hex file nor in any of the hex files I created when I tried mapping the EEPROM.
Since these bytes only hold data when the display is not working, it can very well be the case that this data is part of the problem.

# Conclusion and Next Steps
Any of the four anomalies described above may be the trigger for the timer issue.
However, the question now is: Which one?
Maybe a "wrong" value in the single dose timer area alone triggers the problem.
Maybe more than one EEPRROM area must hold "wrong" values.

To determine which values do trigger the problem without access to the source code of the application it is necessary to modify a vanilla hex file manually and use this for testing.
After each modification it is necessary to test for the timer problem.
The modifications should at least include the following test cases:
-Set the single dose timer to a value greater than ```12C```.
-Set the double dose timer to a value greater than ```1F4```.
-Set the contrast value to a value greater than ```6```.
-Write some arbitrary values to bytes 16 and 17.

So far, I have not conducted these tests.
I will update this post once I performed them.