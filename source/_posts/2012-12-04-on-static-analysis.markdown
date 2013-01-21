---
layout: post
title: "On Static Program Analysis"
date: 2013-01-17 19:28
comments: true
categories: [static program analysis]
tags: [static program analysis, category theory, ruby]
---

#Introduction
As I stated in my [previous post][], my undergraduate thesis is on static
program
analysis of Android applications to prove malicious behavior. In this article,
I try to explain what static program analysis is and isn't and the rational behind using
using it for this research. Oh, and there is code to look at, too!

<!-- more -->

#What is Static Program Analysis
Static Analysis attempts to predict the future behavior of a program. My
[research professor][] has a [great introduction to static analysis][] article
that shies away from a lot of the technical aspects of it.

##Types
There are
two main types of Static Program Analysis: sound and unsound. The difference between
them being that *unsound* is trying to determine behavior based on a partial
modeling of program behavior while *sound* analysis models all behaviors of a
program.
While *unsound* analysis can say something about all lines of code, it will make
assumptions
that certain portions of the program are either unreachable or unimportant for
the analysis, while *sound* analysis does not make these assumptions and tries
to prove and model all behavior.

In other words, *unsound* analysis uses an incomplete model
while *sound* uses a complete model.
Furthermore, *sound* analysis must begin with a soundness proof of all layers of
abstraction. I'll get into more details about this in later articles.

This research is *sound* static program analysis.

##Abstract Interpretation and Soundness
We have a deep and unforgiving failure at the core of the assumption that we can
prove behavior of a program: the [halting problem][] (undecidability). This
ultimately states that it is *impossible* to prove that a program terminates for
all program-input pairs. So, if can't know if a program terminates how can we
determine all paths a program can take? Wouldn't that cause an infinitely deep
tree of paths?

Fortunately, this is only undecidable because in an infinite domain,
we can get around this by presenting a *finite* number of approximations
(abstractions), which guarantees that the abstract interpretation of the program
will terminate.

Let's take the following code
{% codeblock The Never Ending Loop lang:ruby %}
while true
end
{% endcodeblock %}
This is a loop that will never terminate. Graphed like a tree, it would look a
lot like an infinitely long linked list. Graphed like a DFA, if would be a
single state looped back on itself.

However, we can abstract this out into a state that matches a type of infinite
loop.

#Program Verification

##A complete picture
In static analysis, the more information you have about something, the more
likely you'll find a contradiction. The less information you have about
something, the more generalized it is. This relationship comes from Order
Theory.

Rewriting this statement in terms of code, let's give a bunch of information in
a control sequence. Since Ruby hasn't had much love on my blog so far, let's
give it some.

{% codeblock Steal all the passwords lang:ruby %}
if x > 1
    if x < 1
        malware.steal_passwords
    end
end
{% endcodeblock %}
Here we have given enough information to form a contradiction.
$$(x>1)\land (x < 1)$$, so the call to `malware.steal_passwords` can never be
reached.


##Lattices and Partial Order
We can describe relations of information and state by using lattices, which
comes from Order Theory. [An article][], again by my research professor, gives a
good overview of the aspects of Order Theory that pertain to Computer Science.

The basic idea of a lattice in Static Analysis, is that you have a top and a
bottom. At the top, you know so very little information about a *thing* as to
make it generic - like `Object` in `ruby` or `python` are close to the
top. Whereas `x` from the code above is closer to the bottom of the lattice,
having so much information as to cause a contradiction. From this basic idea, we
can define the symbol for a lattice as $$(L,\sqsubseteq)$$ with a supremum $$\top$$ and
a infimum $$\bot$$ (max and min). Where the order of elements is determined *partially*
(asymmetrically).

If you think of a lattice as an upper and lower bound for your program ($$L$$),
then all elements in your program must be partially ordered such that
$$x \sqsubseteq \top$$ and $$\bot \sqsubseteq x$$ $$\forall x\in L$$

As an example:

{% codeblock inheritance lang:ruby %}
class MythicalCreature
  def steal_hearts_and_minds
  end
end

class HippoPony < MythicalCreature
  def steal_souls
  end
end

creature = MythicalCreature.new
ralph = HippoPony.new
{% endcodeblock %}

We can say that $$\text{ralph}\sqsubseteq\text{creature}$$ and that `creature`
is closer to $$\top$$ and `ralph` is closer to $$\bot$$.

This example shows that it is possible to have a provable definition of the order of inheritance that
allows us now to make mathematical assertions and proofs on the inheritance of
that program. We know that all `MythicalCreature` objects can never
`steal_souls` so we can ignore that possibility.

It is one use of a lattice in static program analysis and in a larger context Order Theory.
However, there are
many other ways of using Order Theory to prove things about a program
including *monotonism*, *continuity*, and *fixed points* to name a few.

#Assertions
Our first goal is to have a sound mathematical definition of each element of a
program so that we can assert behavior about that program. Since it is
completely possible to execute a program in theory that could never happen in
practice, it is important to *know* for certain that it really could
*never* happen.

In math, we can prove that $$\sqrt{2}$$ is irrational by contradiction, but we have to make
assumptions like mutually-prime, natural number numerator/denominator pairs are
rational and we must define a strict set of rules for rationality, mathematical
operations and what numbers really are. (If you haven't taken the chance to take
a mathematical analysis class, I encourage you to).

Fortunately for Mathematicians, we *can* and *have* proven these things, so we
can say with certainty that $$\sqrt{2}$$ is irrational. And fortunately for us,
since we have Order Theory, we can prove things about a finite set of elements
that make up a program so that we can make assertions about it, like *this
program will run perfectly 999 times and then start blowing up memory usage*. Or
as I mentioned in my [post on dalvik][], we know that the garbage collector will
never factor into the concerns of the application, because it runs in a separate
environment.

##Security
Security is really a subset of assertions, where the assertions are things like:
*this program cannot access a remote server*, *there is no way for a user to
leave the bounds of the application from within the application*, *there is a
reachable path of execution that will crash the device*

#Caveats
One major issue with static program analysis, is two fold: it needs access to
the source and that source should not be obfuscated.

To be fair, the research that we are doing is on Dalvik byte-code, so we could
disassemble any application. It is also for the military, so it well within the
bounds of operation to know that we will have access to any code that could
potentially be added to a device.

The reason why code obfuscation is an issue, is that there are limitations in
how you can abstractly interpret and ultimately analyse a program. Take my
professor's example of proving that the result of $$4\times -3$$ is negative
without computing it is $$-12$$ and turn it into code:

{% codeblock 4 x -3 lang:ruby %}
->() {4*(-3)}.call
{% endcodeblock %}

We would convert this to $$\alpha(4) \otimes \alpha(-3)$$ which would reduce to
$$\{-\}$$, a negative.

What if we made one of the numbers into a variable?
{% codeblock 4 x -3 lang:ruby %}
four = 4
->() {four*(-3)}.call
{% endcodeblock %}

That is easy enough, we could have easy just substituted in or did an
environment lookup. (For details on that see [substitution][] and
[implementing substitution][]).

What if we make it even harder, by making each number a lambda that creates a
number?
{% codeblock 4 x -3 lang:ruby %}
four = ->(x,y) {x-y+4}
neg_three = ->() {four.call(7, 14)}
->() {four.call(200, (10*20))*neg_three.call}
{% endcodeblock %}

Ok, well, that is easy enough, we could just beta reduce. That would be perfect
except for the small fact that this quickly becomes *undecidable* as the
obfuscation gets worse. (For implementation details of reduction, see my post on [reduction][])

So, code obfuscation has the ability to make it impractical to do static
program analysis.

#Conclusion
The goals of Static Program Analysis lend themselves well to the field of
security. This is particularly true for controlled environments like government.
While it is not a be-all-end-all answer for security, it should allow us to make
strong assertions about the behavior of Andriod applications.

[previous post]: {% post_url 2012-12-01-android-security-with-static-analysis %}
[post on dalvik]: {% post_url 2012-12-01-android-security-with-static-analysis %}
[substitution]: {% post_url 2012-02-05-on-our-way-to-reduction-substitution %}
[reduction]: {% post_url 2012-03-17-reduction %}
[implementing substitution]: {% post_url 2012-02-15-on-our-way-to-reduction-implement-substitution %}
[research professor]: http://matt.might.net
[great introduction to static analysis]: http://matt.might.net/articles/intro-static-analysis/
[halting problem]: http://en.wikipedia.org/wiki/Halting_problem
[An article]: http://matt.might.net/articles/partial-orders/
