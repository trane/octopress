---
layout: post
style: text
title: "Church Encoding: Converting to Primitives"
categories: [lambda calculus, church encoding]
tags: [python, enriched lambda calculus, boolean, numerals, equality, lists]
---

#Introduction

So far, we have been able to build in
[multi-argument
handling]({% post_url 2012-01-14-multi-arg-handling-with-currying %}),
[booleans, numerals, arithmetic]({% post_url 2012-01-14-church-encoding %}),
[equality and lists]({% post_url 2012-01-15-more-church-encoding %})
all while staying in the Pure λ-Calculus. However, it is convenient to introduce
primitives like numbers and booleans when working on more complicated examples
in order to remove some extra cognitive steps. As an examples, which takes fewer
steps to recognize: `2` or the return value of `scc(scc(0))`?

In this post, I describe the means of converting some of the strictly pure
λ-calculus primitives to a more common numeric and boolean representations that
Pierce calls λNB - which is his name for the *enriched* λ-calculus. I will also
detail how to convert the other way from λNB→λ.

#Boolean

As a reminder, the Church boolean `tru` and `fls` are defined:
{% highlight text %}
tru = λt. λf. t
fls = λt. λf. f
{% endhighlight %}

##Church boolean → Boolean

To convert from λ→λNB we simply apply the λ-expression to `true` and `false`:
{% highlight text %}
realbool = λb. b true false
{% endhighlight %}
{% highlight python%}
# realbool = λb. b true false
realbool = lambda b: (b)(True)(False)

>>> realbool(tru)
True
>>> realbool(fls)
False
{% endhighlight %}

##Boolean → Church boolean
In the other direction, λNB→λ, we use an `if` expression:

{% highlight text %}
churchbool = λb. if b then tru else fls
{% endhighlight %}
{% highlight python%}
# churchbool = λb. if b then tru else fls
churchbool = lambda b: (tru) if b else (fls)
>>> realbool(churchbool(False))
False
>>> realbool(churchbool(True))
True
{% endhighlight %}

#Equality

Just like we were able to build higher level equality functions using Church
booleans, we can do higher level conversions as well. As a reminder of the
definition of equality:
{% highlight text %}
equal = λm. λn. and (iszro (m prd n))(iszro (n prd m))
{% endhighlight %}

##Church equality → Equality
{% highlight text %}
realeq = λm. λn. (equal m n) true false
{% endhighlight %}
{% highlight python %}
# realeq = λm. λn. (equal m n) true false
realeq = lambda m: lambda n: (equal(m)(n))(True)(False)

>>> realeq(c1)(c1)
True
>>> realeq(c1)(c2)
False
{% endhighlight %}
##Equality → Church equality
{% highlight text %}
churcheq = λm. λn. if equal m n then tru else fls
{% endhighlight %}
{% highlight python %}
# churcheq = λm. λn. if equal m n then tru else fls
churcheq = lambda m: lambda n: (tru) if m == n else (fls)

>>> realbool(churcheq(3)(3))
True
>>> realbool(churcheq(3)(2))
False
{% endhighlight %}

#Numbers

I have already been converting Church numerals to numbers in my previous posts
using the `(lambda n: n+1)(0)` function, here we will define it as a callable
function. Church numerals are defined as:
{% highlight text %}
scc = λn. λs. λz. s (n s z)
c0 = λs. λz. z
c1 = scc c0
c2 = scc c1
c3 = scc c2
{% endhighlight %}
##Church numeral → Natural number
{% highlight text %}
realnat = λm. m (λx. succ x) 0
{% endhighlight %}
{% highlight python %}
# realnat = λm. m (λn. succ n) 0
realnat = lambda m: (m)(lambda n: n + 1)(0)

>>> realnat(c2)
2
>>> realnat(times(c2)(c3))
6
{% endhighlight %}
##Natural number → Church numeral
This is more complicated and possible conversions require methods that I have
not covered yet. I will revisit this when I post about Recursion.
