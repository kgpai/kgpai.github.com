---
layout: post
title: "Clean Code Notes"
description: ""
category: 
tags: []
---
{% include JB/setup %}


Clean Code: A Handbook of Agile Software Craftsmanship
======================================================
These are some of my notes from [Clean Code - A Handbook of Agile Software Craftsmanship] (http://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882). The book aims 
to shed some light on some of the non engineering aspects of writing code. When I write code, I spend an inordinate amount of time on intangibles like the name of a variable, the structure of 
a class, the visiblity of certain functions, etc; These things are hard to get right, but its importance cannot be underestimated by any software craftsman - If not for the joy of working over a beautifully structured and crafted code base , the long term the cost of maintaining elegant code pays itself many times over. I am thus reproducing some of the more salient points of this book, and I would highly recommend that you go get a copy for yourself. 


Chapter 1 
-------------
 * Bad code slows you down and is very very expensive. Bad code doesn't necessarily mean non functional or buggy code, just code that is hard to understand or modify. 
 * LeBlanc's Law: Later is never. 
 * Problem with messy code, is that no change is trivial and every change needs to be properly understood along all the tangles, twists and knots of the system. Productivity declines , and this leads to further pressures to add features fast, which leads to more short cuts ( Hey the system is a mess anyway! ), which leads to more bugs and slower productivity and this spiral's out of control. Eventually this leads to a 'rewrite'. 
 * Programmers must always defend the code with passion. Managers want good code, even if they obsesses about the schedule. 
 * As a programmer you will not make the deadline by making a mess, the only way to go fast and keep momentum is by writing the code as clean as possible at all times. 
 * It is however hard to know when a code is clean and when it is not. You need a 'code-sense' which like is talent is present in large quantities in the great programmers, for the rest of us mortals its something that needs to be developed and nurtured. 
 * The ratio of time reading code to writing code is 10:1 , thus code should always be structured so its easy to read than write. 
 * Apply the boy scout rule to code: 'Leave the campground cleaner than you found it'. Code has to be kept clean over time !  


 