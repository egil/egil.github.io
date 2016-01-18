---
layout: post
title: Design by Contract should become this decades TDD
created: 1318572037
redirect_from:
  - /2011/10/13/design-contract-should-become-decades-tdd/
---
I am currently taking CSE 218 at UCSD, "Advanced Topics in Software Engineering - Methods and Tools for Software Modularity", where we recently read the paper ["Applying 'Design by Contract'"](http://ieeexplore.ieee.org/xpl/freeabs_all.jsp?arnumber=161279) by Bertrand Meyer. I already knew a little about the concept, and after reading that paper and playing with the concept in a micro project, I am sold.  I really think Design by Contract has a chance at being this decades TDD. It is pretty clear to me why it is a good idea to do "Contract First" development. Let me try to explain why.

<!--break-->

For me, at least, when developing a new class or module, no matter if there is a detailed interface contract or design document in place or I am gong-ho-agile-styling it, it is an iterative process, where I add one bit of functionality after the other as I go. Most of the time, unless it's just a simple utility library, there is state involved, and at the time I add the local variable to hold a state, I also find myself trying to restrict that variable as much as possible with various parameters, modifiers, checks, encapsulation, etc., to prevent the state of the object from going bad. That process of preventing bad state never really stops until all functionality that deals with that state is added, which is tedious and error prone.

Enter code contracts. When I declare my local state variable, I also declare my class invariants for it, in the same way we create unit tests to verify our code, before I start adding functionality to manipulate the state. The benefits are obvious to me:

- With good tooling, I get compile time suggestions about what methods and properties might break my invariants as I continue to develop my class.
- The maintainability of my code just sky-rocked. Another developer can pick up the code and do his modifications, without worry that he might break an implicit contract.
- The code that is much more self-documenting. Again, with proper tooling, contracts can be extracted and presented to the users, there is basically no hidden information left in the code anymore (e.g. comments like "// must not be zero!"). And documentation doesn't become stale.

The downside is that you have to define those invariants (and requires/ensures), just like you have to create your tests in TDD.

Several things were brought up in the discussion in class. Somebody asked why this is not more widely used in the industry. I think the answer is that tooling for the most widely used languages (C, C++, Java, C#) are just starting to get there. The amazing thing is that, compared to TDD, the benefits of the effort of adding contracts to our code will continue to grow as new tools become available. Automated bug finding tools would probably be much better if they could also consider contracts. I can also see tools for finding security vulnerabilities reaching a higher level, being able to detect more complex security exploits, with less false positives, if they could draw on contracts to do their analysis. And when a updated or new tools is available, we can just run the tool on our existing code and instantly get the benefits it provides.

It was also mentioned by somebody with personal experience that it is hard to add contract to existing systems, and I can indeed imagine it being very hard to an existing codebase, especially one that is large and complex. It is probably, just like with unit tests, much harder to add contracts as an afterthought than it is to do it up front, so contracts might be too costly to add to existing systems. For new systems, it is hard for me to argue against design by contracts.

For more info and links to tools (I played with Code Contracts from Microsoft), I recommend the [Design by Contracts page](http://en.wikipedia.org/wiki/Design_by_contract) on Wikipedia.
