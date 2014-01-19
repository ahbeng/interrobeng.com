---
layout: post
title: "NUS Matriculation Number Check Digit Algorithm"
date: 2014-01-19 13:02:33 +0800
comments: true
categories: NUS
---
### Summary

A brief discussion of the [check digit](http://en.wikipedia.org/wiki/Check_digit) algorithm for matriculation numbers / NUSNET IDs prefixed by either U or A, accompanied by a JavaScript implementation and a client-side [calculator](/2014/01/19/nus-matriculation-number-calculator/).

Initiated due to a collaboration with [Camillus](https://www.qxcg.net/) on a bookmarklet which involved mapping NUSNET IDs to matriculation numbers for use with a [redacted] endpoint - I initially helped with the U prefix case, as it is more familiar to old farts like me.

### Related Work

Various equivalent forms of the algorithm have been discussed under [NUS Matriculation Number Checkdigit](http://musingsofanaspiringpolymath.blogspot.sg/2008/06/nus-matriculation-number-checkdigit.html), [NUS Matriculation Number Checkdigit 2](http://musingsofanaspiringpolymath.blogspot.sg/2013/03/nus-matriculation-number-checkdigit-2.html) and [Validate Your NUS Undergraduate Matric Number](http://phyublog.blogspot.sg/2009/11/validate-your-matric-number.html). Also, there is a server-side [NUS Matric Resolver](http://nvquanghuy.com/matric/) by [Huy Nguyen](http://nvquanghuy.com/) and a [Matric.js](https://gist.github.com/lowjoel/6328287) gist by [Joel Low](http://joelsplace.sg/).

Though correct, most of them involve extraneous steps, especially for U-prefixed matriculation numbers. It kinda bugged me, so I worked out a more elegant formulation during a short plane ride last week.

### Algorithm

1. For U-prefixed NUSNET IDs such as `u0906931`, discard the third digit - u09<del style="color:red">0</del>6931, resulting in `u096931`, which is the corresponding matriculation number without its check digit.

2. Let the last 6 digits be d<sub>1</sub> to d<sub>6</sub>. Compute the weighted sum s = w<sub>1</sub> &times; d<sub>1</sub> + ... + w<sub>6</sub> &times; d<sub>6</sub> using the corresponding weights:

   Prefix|w<sub>1</sub>|w<sub>2</sub>|w<sub>3</sub>|w<sub>4</sub>|w<sub>5</sub>|w<sub>6</sub>
   :-:|:-:|:-:|:-:|:-:|:-:|:-:
   **U**|0|1|3|1|2|7
   **A**|1|1|1|1|1|1

3. Find check digit corresponding to the remainder of the weighted sum divided by 13 (sum modulo 13):

   Remainder|0|1|2|3|4|5|6|7|8|9|10|11|12
   :-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:
   **Check Digit**|Y|X|W|U|R|N|M|L|J|H|E|A|B

### Notes

The weights for A-prefixed matriculation numbers are all 1, i.e. the weighted sum is simply equal to the sum of the digits themselves, so their specific algorithm could be simplified further. However, it is formulated like this to unify the algorithms for both current prefixes, as well as possibly accommodate future prefixes with different weights.

If this scheme and blog post are somehow still surviving in the far future, you would have to use all 7 digits instead of the last 6, when there have been over 1 million NUS undergraduates since 2010.

**Historical note:** The U prefix applied to undergraduates from AY2009/10 and before; the A prefix applies to undergraduates from AY2010/11 and after.

### JavaScript Implementation

{% gist_no_css 8503349 matric.js %}

### Matriculation Number Calculator

Could come in handy for quickly finding project mates' matriculation numbers.

{% jsfiddle u8X5W result,js,resources,html,css default 400px %}
