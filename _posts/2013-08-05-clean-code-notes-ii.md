---
layout: post
title: "Clean Code Notes II"
description: ""
category: posts 
tags: [code, software engineering, java ]
---
{% include JB/setup %}

Chapter 2
-------------
"The hardest problem in computer science is naming"

* Use intention revealing names. Choosing names that reveal intent can make it
much easier to understand and change code. For eg:
   <pre><code>
    int d; //elapsed time in days
    //should be changed to 
    int elapsedTimeInDays;
    </code> </pre>

* Avoid Disinformation :
  + Do not use names which have other predefined meanings . 
  + Do not use names loosely for eg if you have a set of accounts, do not call
    it account list etc. 
  + Beware of names which differ in small , subtle ways. 


* Make meaningful Distinctions :
  + Avoid number series names : a1, a2 etc..
  + Avoid noise words which do not add extra information for e.g : class
    ProductInfo or Class ProductData - different names but hard to infer any
    distinction in purpose. 
  + The above especially applies to function names , for e.g
    `getActiveAccount()`, `getActiveAccountInfo()` etc. 

* Use Pronounceable Names : Hungarian notation is an abomination that should be avoided. 

* Use Searchable Names : If something is significant give it a proportionally
significant name. The length of a name should correspond to the size of its
scope. If a variable or constant might be seen or used in multiple places in a
body of code, it is imperative to give it a search friendly name. 

* Do not prefix member variables with m_ anymore. Classes and functions should
be small enough that you don't need them. Prefixes just add clutter and are
easily ignored. 

* It is unnecessary to adorn your interfaces and implementation - Leave the
interface unadorned and encode the implementation ; e.g ShapeFactory and
ShapeFactoryImpl. 

* Class Names: Classes and objects should have noun or noun phrase names like
Customer, WikiPage etc. Avoid words like Manager, Processor, Data, Info
etc. Above all a class name should not be a verb. 

* Method Names
  + Methods should have verb or verb phrase names like render, deleteWorkItem.
  + Accessors, mutators and predicates should be named for their value and
  prefixed with get, set and is according to  the javabean standard. 
  + When constructors are overloaded use static factory methods with names that
  describe the arguments. 
  Complex invectionPoint = Complex.FromRealNumber(12.0); 
  + Consider enforcing their use by making corresponding constructors private. 
  + Pick one word per concept - dont have get,fetch,retrieve in a
 class. Similarly dont have Manager,Controller,Driver all at once. Stick to
 one. A consistent lexicon is a very very nice thing.  
 + Use Solution Domain Names : Use cs specific terms, algorithm names, pattern
 names, math terms etc whenever possible. 
 + Use Problem Domain Names : When ever there is no such programmeresse , use
  names that correspond do the problem domain. 

Chapter 3
------------------
"Functions"

* "Small!": Functions should be small , as small as possible. How small ? A
function should not exceed more than 10 lines at worst and even  then look at
possibilities to break it up. 
* Do not keep the indent level of a function greater than two - This helps in
readability and debugging. 
* Do one thing : Functions should do one thing. They should do it well. They
should do it only. 
* It is hard to know what this 'one' thing is. 
* Decompose a higher level function into an abstraction one level below it. 

* A function should then be describable as a brief TO paragraph:
  + TO name_of_function , we do thingummyjig while checking for
  thingummybob. Then we check in our thinglist for result of thingummyjig
  computation, while returning a boolean if found or not.

* Again, the reason to write functions is to decompose a larger concept into a
set of steps at the next level of abstraction. 

* The Stepdown Rule: Code should read like a top-down narrative. i.e You
should be able to read the program as though it were a set of TO paragraphs
each o which is describing the current level of abstraction and referencing
subsequent TO paragraphs at the next level down. 
  + To include the setups and teardowns, we include setups, then we include the
  test page content, and then we include the teardowns.
  + To include the setups, we include the suite setup if this is a suite, then we
  include the regular setup.
  + To include the suite setup, we search the parent hierarchy for the
  “SuiteSetUp” page and add an include statement with the path of that page.

* Switch Statements: Switch statements by nature will not be small and will
always do N things. You can't always avoid switch statements, but you can
ensure that they are buried in low level classes and are never repeated. One
way to do this is by appropriately using Polymorphism. Note: That this is a
subjective call, always prefer composition to inheritance so switch to
polymorphism only when the use case supports creating polymorphic objects and
is hidden behind an inheritance relationship.

* Use Descriptive names: Modern IDE's make it trivial to change names, so
experiment till you find the right one. Do not be afraid of long names if they
are descriptive and describe the function precisely. 

* Function Arguments :
  + The ideal number of arguments for a function is zero (niladic).
  + Next comes one (monadic), followed closely by two (dyadic). Three arguments
(triadic) should be avoided where possible and more than three (polyadic)
requires special justification - and even then shouldnt be used anyway. 
  + Arguments are generally at a different level of abstraction compared to the
  function. 
  + Increasing arguments also exponentially increase testing effort. 
  + Never mix output and input arguments - Output arguments are also harder to
  understand than input arguments. 

* Common Monadic Forms: 
  + Asking a question about an argument as in boolean fileExists("myfile") . 
  + Second is operate on an argument, transform it and then subsequently
  returning it, for eg File open("myfile"). 
  + A somewhat less common form is an 'event' which has no output. The overall
  program is meant to interpret the function call as an event and use the
  argument to alter the state of the system. For e.g void
  passwordAttemptFailedNtimes(int numAttempts).

* Dyadic Functions:
  + These are naturally harder to understand than monadic functions, since it
  isnt  implicitly clear how the arguments are related to the function. Also it
  is human nature to skip some of the arguments when reading code. E.g
  writeField(outputStream, name). Note functions like Point(0,0) even though
  diadic in form are actually monadic in nature. Again while it is not possible
  to avoid dyadic forms it is always imperative that these be kept at a
  minimum. 

* Triads - These are significantly harder to understand compared to diadic
functions because of ordering issues, testing, etc. E.g consider a function
like assertEquals(message, expected, actual). This will require some scrutiny
from the programmer if he is not to make an error. 

* Have No Side Effects : As mentioned earlier functions should do one thing and
do one thing well. Functions that have side effects in effect then are doing
hidden things. This can lead to temporal couplings , order dependencies, race
conditions etc. As far as possible design your functions so that they are
functional in nature and have no side effects. 

* Output Arguments: Output arguements are arguments that are to be output (e.g
appendFooter(s) ). In effect these merely add the argument to an 'output'. These should
again be generally avoided - again if function must change state it should do
so the the owning object. 

* Command Query Seperation: Functions should either perform a Command or Answer
a Query never do both. Functions that do this again go against the 'do one
thing' principle. A bad example of a function like this is : ` public boolean set(String attribute, String value); `

* Avoid Error Codes : Not only is it an artifact of low level C code, it also
 violates command query seperation. Prefer exceptions to error codes always. An
 example of a bad pattern is : 
 <pre> <code>
 if (deletePage(page) == E_OK) {
  if (registry.deleteReference(page.name) == E_OK) {
     if (configKeys.deleteKey(page.name.makeKey()) == E_OK){
     logger.log("page deleted");
     } else {
     logger.log("configKey not deleted");
     }
  } else {
    logger.log("deleteReference from registry failed");
 } </code> </pre>

* A corlloary to this is to extract your try/catch blocks into seperate
  functions - generally in an encompassing function so that your code isn't
  littered with try/catch blocks. 
* Error Handling is One Thing !: Function that handles errors should do
  nothing else. This means that if the keyword try exists in a function, it
  should be the very first word in the function and there should be nothing
  after the catch/finally blocks. 

* The Error.java Dependency Magnet : Usually every large project ends up
  having an enum like so : 
 <pre> <code> 
 public enum Error {
 OK,
 INVALID,
 NO_SUCH,
 LOCKED,
 OUT_OF_RESOURCES,
 WAITING_FOR_EVENT;
 }
 </code> </pre>
 Classes like these are a dependency magnet; since many other classes must
 import and use them. A change here will affect every other package that uses
 this class - This puts a reverse pressure from modifying the Error
 class since now you have to redeploy these changes to all other dependent packages. Avoid situations which can lead to these kinds of dependency
 pressures. When exceptions are used, you can derive new exceptions without
 forcing any recompilation/deployment. 

* Dont repeat yourself : It is all to easy to duplicate code, and code
  structures. Refactor often when you sense that code is being duplicated
  in or across function boundaries. 

* Iterate multiple times to finally get the code right: It is not possible to
  write code that fits all the above constraints in the first attempt. Iterate
  and refine succesively till you feel comfortable. 
* Always write unit tests to cover your code so that it can be tested as you
  refactor. 




 





