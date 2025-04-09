---
layout: single
title:  "Teensy Trouble"
date:   2019-01-09 22:50:00 -0600
categories: posts
use_math: false
header:
  # overlay_image: /assets/images/barycentric/barycentric-colors-blackfill.png
  overlay_image: https://www.pjrc.com/store/teensy36.jpg
  overlay_color: "#000"
  overlay_filter: "0.3"
  caption: "Photo credit: [**PJRC**](https://www.pjrc.com)"
  og_image: https://www.pjrc.com/store/teensy36.jpg
  teaser: https://www.pjrc.com/store/teensy36.jpg
tags:
  - microcontrollers
  - hardware
toc: false
---

In my Ph.D. studies, I have been doing a fair amount of embedded programming.
My favorite microcontroller by far has been the aptly-named [Teensy](http://www.pjrc.com/teensy).
These microcontrollers have some serious hardware at a very reasonable price.
They are also incredibly easy to program due to the fact that they can be programmed with arduino code.
I could go on and on about why I like them, but today I found a relatively minor issue that caused me a lot of trouble.

The Teensy 3.6 has many pins capable of reading analog voltages.
The two pins I decided to use for my project are pins 32 (A13) and 35 (A16).
I implemented the functionality using `analogRead(13)` and `analogRead(16)`, respectively.
Interestingly, A13 worked as expected, but A16 did not.
Instead, it behaved like a floating pin, and even shorting that pin to ground had no impact on the reading (this should have been a clue, actually).
It wasn't just A16, either.
All analog pins numbered A14 or higher were not working, while all numbered A13 or lower were working as expected.

After much frustration and much investigation, I found the answer.
Since `analogRead()` can accept either analog pin numbers or actual pin numbers as its argument, numbers over 14 are ambiguous to the code, so it assumes the argument is the actual pin number.
To avoid this, you can either use the actual pin number or use the analog pin number preceded by "A" (i.e. A14, A15, etc.).
This works because those are just macros to the actual pin numbers.
In my case, using `analogRead(A16)` solved the problem.