---
layout: post
title:  "Why I love TDD"
date:   2015-04-03
categories: TDD testing
---


TDD or Test-Driven Development as the name same says, is to develop with the wise guidance of testing.

Many only write the tests, but don't allow the testing process to guide them. If you do this, you're the right path!
But only writing tests for assurance is not using the most benefitial part of the TDD process, which is simple and
evolutionary design.

##Write your tests first

I'm still talking about leaving the design of your programs to the testing process. And the first step to do it is
start to writing your tests first. Need a new object to be shown on a [Django] template? The first thing you should
do is write a test to it. You could argue that this is too simple to test? Yes, in this case we'll mostly test it for
assurance, *because it's too simple*. If it was more complicated, we'd be testing for the design too.

##A not so simple example: Loan Calculator

I'm not going into the implementation details because they can be overwhelming at a superficial view.
Instead, here's the main problem we face when implementing it: there are a lot of calculations to get to
the key formula parameters. So we have some key formulas: calculating the installments and its principal value
and interest value (of each installment). And we know that we have to calculate the intermediate values to apply
the formula to them.

Looking at it at distance, its easy enough to perform an imaginary implementation cenary. But by not knowing the loan
processes well, you may find your future self stuck when trying to support different kinds of loans.
But we don't wanna think about that now, eXtreme Programming is all about simplicity, so let's keep it simple.

Not caring about the future at all, let's exercise the possible implementations.

**Possibility 1**: Create main functions (on a class?) for the key formulas. Calculate the intermediate values in-place.

**Possibility 2**: Create main public functions for the key formulas, create private functions for intermediate values.

What should we choose?
I won't tell you :P

No, really, I won't. Instead we're going to figure it out by going through the testing process, using our imagination.

As I said before, the first thing you should always do is write a test.

Lets stress a bit what our first test should do. We probably wanna start with the key formulas first, because that's
our final destination. The next question is, which of the key formulas is the easiest? For this we need to avaliate
the parameters of each formula and the intermediate calculations it performs, you don't need to go too deep into that
analysis, just look at the formulas and use your gut feeling to find out which should be easier to implement.
We'll choose the total amount of a installment to start for no specific reason.

Ok, now we know where to start and I'll pretend we all know the formula and its intermediates calculations.

But we still don't know what our first test is going to be. We need to ask ourselves more questions.
What is the easiest piece of this calculation that I can begin with? We know that the key formula has some intermediate
calculations, how about we start with that?

###One step closer

Now we decided to break the problem again and test an intermediate calculation before getting to the
key formula. But there are at least 4 intermediate calculations to go through. Which one should we test?
Again, we should ask ourselves: which one is the easier to implement? And again use your gut to find out which one is easier.


So now we wrote an imaginary test to the easiest of our 4 intermediate calculations for one of the key formulas we picked.

Now what? You may not realized it, or I maybe written it in a really boring way, but we just got through the most important
part of TDD. After that comes the very well documented TDD red-green-refactor-repeat cicle. This whole text until now was
about how to arrive in the red, which is the first step of the cicle.

##Green

The green step should be the easiest, since we've been picking the easiest pieces to code through our whole thinking process.
If the implemenation (green) step is too hard, it means you didn't picked the easiest piece of the problem to solve and you
need to go back one step and change something. Repeat this until you have some easy to implement piece of code. Sometimes this
can be hard to do, when it gets hard, try looking at the problem from a higher ground, sometimes we need to look at the
problem from a different perspective to get it right.

##It should always be easy.

For me this is the greatest benefit of TDD. It has the power to transform a really hard problem into little really easy
ones. This may be embaracing for some, but I can't come up with a recursive solution without TDD. And I have tried.
It just comes up naturally when I'm going through the TDD process. A very good way to understand better the TDD benefits
is to do a coding Dojo with a recursive problem. I recommend the [Word Wrapping](http://www.codingdojo.org/cgi-bin/index.pl?KataWordWrap) problem. After you solve it, read
the [Uncle Bob Martin article](http://thecleancoder.blogspot.nl/2010/10/craftsman-62-dark-path.html) about it. This was one of the things that opened my eyes a little more about TDD.

If you made it down here, thanks for the patience. I still don't have comments enabled, but as the git user I bet you are you
can comment on the [source code of this post](https://github.com/flaviamissi/flaviamissi.github.io/commit/c68d127b07ba30444025fd0474a7ee14cb199eab) :)
