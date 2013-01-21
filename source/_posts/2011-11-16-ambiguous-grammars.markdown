---
layout: post
style: text
title: Ambiguous Context-Free Grammars
categories: parsing
---

#Introduction

In this post, I am going to describe what makes a Context-Free Grammar (CFG)
ambiguous. I'm doing this because I have a test tomorrow on Pushdown Automata
(NPDA), CFG, Pumping Lemma, and (Non)?Deterministic Turing Machines - what
better way to study than this?

#Ambiguity

A grammar is considered ambiguous simple if it has more than one way that it can
construct a parse tree for the same input. That's it! A simple example is in
a CFG for this a math expression parser:
{% highlight ruby %}
# a+a*a
S -> S + S | S * S | (S) | a
# or expanded
S -> S + S
S -> S * S
S -> (S)
S -> a
{% endhighlight %}
So, why is this ambiguous? For most of us we immediately see that we should
multiply first and then add, per the order-of-operations rule. However, when
building parse trees off of that CFG, we can see that it can be made into two
different trees:
{% highlight ruby %}
    S                S
    |                |
  S + S            S * S
 /  |  \          /  |  \
a   + S * S    S + S *   a
      | | |    | | |
      a * a    a + a
{% endhighlight %}
The key to remember is that we *know* that the multiplication should come first,
so we should fix this.

#Disambiguate

The part that makes this grammar split is that we are referencing addition,
multiplication, and terminal *a* with the same Non-terminal *S*. There is no
algorithm that can tell if a CFG is ambiguous, however there are some techniques
for disambiguating:
* Add non-terminals
* Divide into: Expressions, Factors, Terms

{% highlight ruby %}
# a+a*a
S -> E
E -> E + T | T # E is an expression
T -> T * F | F # T is a term that disambiguates * and +
F -> (E) | a   # F is a factor, isn't affected by * or +
# compacted
S -> S + T | T
T -> T * F | F
F -> (S) | a
# or expanded
S -> E
E -> E + T
E -> T
T -> T * F
T -> F
F -> (E)
F -> a
{% endhighlight %}
h
This will ensure that we end up with a single parse tree for that string.
{% highlight ruby %}
    S
    |
  S + T
 /  |  \
T   + T * F
|     | | |
F     a * a
|
a
{% endhighlight %}

#Inherent Ambiguity

Inherently ambiguous Context-Free *Languages* are languages that have no
unambiguous CFG. This is a tough one to determine because not only is there no
algorithm to help, but you must prove that it is ambiguous *for all* grammars.
Wikipedia has a [good
example](http://en.wikipedia.org/wiki/Ambiguous_grammar#Inherently_ambiguous_languages)
of an inherently ambiguous language. Though, it is sort of cheating, since the
language defined by the union of the two CFLs is *not* a CFL. I prefer the CFL:
$$a^ib^jc^k \vert i,j,k \ge 0 \land i = j \lor j = k$$

