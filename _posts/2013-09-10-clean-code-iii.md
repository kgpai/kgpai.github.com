---
layout: post
title: "Clean Code III"
description: ""
category: 
tags: [code, software engineering, java, comments]
---
{% include JB/setup %}

Chapter IV
-----------
"Comments"

* Comments are a necessary evil. They are a result of the unexpressiveness of the programming language or the programmer unable to properly articulate the idea in code. 
* If you ever feel the need to write a comment,  see if its absolutely not possible to express yourself in code. 
* Comments need to be maintained just as code is maintained. In some ways comments are harder to maintain since there is no formal relationship between the code and the comment that describes it. As the code changes its easy to overlook the corresponding comment leading to a flase comment at best and a source of confusion and bugs at worst. 
* Ideally code should be expressive and clear enough so that it does not need comments in the first place. 

* Comments do not make up for bad code. Instead of writing comments to explain bad code, clean it up. 

* Some comments are beneficial. Keep in mind though, the only good comment is the comment you found a way not to write. 



Chapter V
----------
"Formatting"
* Impressions matter: Poorly formatted code gives the impression that the same level detail went into writing the code. 
* Nearly all code is read left to right and top to bottom. Each line represents an expression or a clause, and each group of lines represents a complete thought. Those thoughts should be separated from each other with blank lines. 
* Vertical density implies close association. So lines of code that are tightly related should appear vertically dense. Often comments appear where they shouldnt and destroy this association. 
* The same point above applies to closely related concepts. They should be kept close together so that you do not have to jump between files trying to understand what a system is doing. This is one reason why protected variables should be avoided. (Since their scope is visible but restricted inside a lot of child classes).
