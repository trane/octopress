---
layout: post
style: text
title: More on Ambiguous CFGs
categores: parsing
---

#Introduction

In my previous post, I described some ambiguous CFGs for the simple math
expression: a+a\*a. In this post, I'll be talking about a more complicated, but
not more difficult ambiguous context-free grammar, the if-else-then context-free
language.

#More complex example
Take this grammar describing *if q then if q then p else p*:
{% highlight ruby %}
# if q then if q then p else p
S -> if E then S | if E then S else S | P
P -> p
E -> q
{% endhighlight %}
Why is this an ambiguous grammar, well, it has to do with the *else* statement.
*else p* can be the end of the inner *if q then p* or the end of the outer *if q
then if q then p*. Let's write out the parse trees to make some visual sense.
{% highlight ruby %}
# if q then if q then p else p
       S                    S
       |                    |
if E then S else S      if E then S
   |      |       \        |      |
   q  if E then S  P       q   if E then S else S
          |     |  |              |      |      |
          q     P  p              q      P      P
                |                        |      |
                p                        p      p

{% endhighlight %}

#Disambiguating Strategy
##Cut into smaller modules

If you think of the grammar as being cut up into smaller named modules, it makes
it easier to abstract away the more complex strings generated. For example, *if
E then S* we could call *U* (for unmatched else) and *if E then S else S* we
could call *M* (for matched else).
{% highlight ruby %}
S -> U | M
U -> if E then S
M -> if E then S else S
{% endhighlight %}
##Create terminating, self-referencing only statement

Then we begin to disambiguate the grammar by removing the some inner
relationships between *S*, *U*, and *M*. Start by making *M* only
self-referential and terminaling.
{% highlight ruby %}
M -> if E then M else M | P
{% endhighlight %}
##Make rest of non-terminals, tail-recursing and set your base-case

Then we make *U* the unbalanced else by tail-recursing and referring to *S* in
order to eventually terminate.
{% highlight ruby %}
U -> if E then S          # base case
     | if E then M else U # recurse
{% endhighlight %}

#Unambiguous Grammar

Now we have an unambiguous grammar, ensuring that all matched *else* strings
follow one branch and unmatched *else* strings follow another.
{% highlight ruby %}
S -> U | M
U -> if E then S          # base case
     | if E then M else U # recurse
M -> if E then M else M | P
P -> p
E -> e
# if q then if q then p else p
    S
    |
    U
    |
if E then S
   |      |
   q      M
          |
    if E then M else M
       |      |      |
       q      P      P
              |      |
              p      p
{% endhighlight %}

