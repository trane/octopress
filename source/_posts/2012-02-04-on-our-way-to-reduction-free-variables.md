---
layout: post
title: "On Our Way to Reduction: Free Variables"
categories: [lambda calculus, reduction, set theory]
tags: [lambda calculus, proof, set theory]
---

##Introduction
The ability to reduce is one of the key components to the λ-calculus. However,
you find it directly implemented in compiler optimizations for both functional
and imperative languages and as inspiration in
[Google's MapReduce](http://en.wikipedia.org/wiki/MapReduce).

However, there are some subtleties that must be addressed to go from our λ
and λNB calculi to being able to implement reduction. In this post, I
mathematically prove that reduction on the λ-calculus terms can be done.

##Define The λ-Calculus
As I described in my [initial post]({% post_url 2012-01-10-the-lambda-calculus %})
about the λ-calculus, it is a *very* simple definition:
{% highlight text %}
t ::=           terms:
    x         variable
    λx.t   abstraction
    t t    application
{% endhighlight %}

##Definition of a Set of Free Variables and Size
Free variables are defined as:
{% highlight text %}
FV(x)       = {x}
FV(λx.t1)   = FV(t1) \ {x}
FV(t1 t2)   = FV(t1) ∪ FV(t2)
{% endhighlight %}

Size is defined as:
{% highlight text %}
size(true)                  = 1
size(false)                 = 1
size(0)                     = 1
size(succ t1)               = size(t1) + 1
size(pred t1)               = size(t1) + 1
size(iszero t1)             = size(t1) + 1
size(if t1 then t2 else t3) = size(t1) + size(t2) + size(t3) + 1
{% endhighlight %}

##Proof that |FV(t)| &#8804; size(t)
Unlike many statements in math, intuition and reality are in sync on this
statement. Of course the statement `{ ∀ t, FV(t) ≤ size(t) }` is true. However,
it is important to formally prove this, since this is key to reduction.

By induction, by proving the following three cases, we can prove the statement
to be true for all `t`:

###Case 1:
{% highlight text %}
t = x
|FV(t)| = |{x}| = 1 = size(t)
{% endhighlight %}
###Case 2:
{% highlight text %}
t = λx.t1
Inductively:
|FV(t1)| ≤ size(t1)
So:
|FV(t)| = |FV(t1) \ {x}| ≤ |FV(t1)| ≤ size(t1) < size(t)
{% endhighlight %}
###Case 3:
{% highlight text %}
t = t1 t2
Inductively:
|FV(t1)| ≤ size(t1) and |FV(t2)| ≤ size(t2)
So:
|FV(t)| = |FV(t1) ∪ FV(t2)| ≤ |FV(t1)| + |FV(t2)| ≤ size(t1) + size(t2) < size(t)
{% endhighlight %}

