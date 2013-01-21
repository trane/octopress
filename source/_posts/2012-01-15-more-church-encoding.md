---
layout: post
style: text
title: "Church Encoding: (In)Equality and Lists"
categories: [lambda calculus, church encoding]
tags: [python, lambda, equality, lists]
---

#Introduction

In my
[previous post]({% post_url 2012-01-14-church-encoding %})
about Church Encoding, I built up
some booleans and their respective operators, numerals and several arithmetic
operators. This post will focus on building two important constructs for any
programming language: equality testing and lists. I will also
continue my implementation with Python.

#Testing For Zero

Determining if a Church numeral is zero is done by finding a pair of arguments
that will return whether the numeral is zero or not (a True/False expression).
We can use the `zz` and `ss` terms from the
[subtraction]({% post_url 2012-01-14-church-encoding %})
operation, by applying our numeral to the
pair `ss` and `zz`. The trick is, if `ss` is applied at all to `zz` we know that
the numeral is not zero and return `fls`, otherwise we return `tru`. This makes
perfect sense, since the numeral will be applied the number of times equal to
its value.

In other words,
if the Church numeral is `0`, then `ss` will be applied `0` times to `zz` and
will return `tru`. Once `ss` is applied to `zz` (if the numeral is not `0`), it
will return `fls` - not `0`.

{% highlight text %}
iszro = λm. m (λx. fls) tru
{% endhighlight %}

{% highlight python %}
# iszro = λm. m (λx. fls) tru
iszro = lambda m: (m)(lambda x: fls)(tru)

iszro(c1)(True)(False)
False
>>> iszro(c0)(True)(False)
True
>>> iszro(sub(c3)(c3))(True)(False)
True
>>> iszro(sub(c3)(c2))(True)(False)
False
>>> iszro(times(c0)(c1))(True)(False)
True
{% endhighlight %}

#Numeric Equality

There are probably many ways to define numeric equality, however, the trick I
will use is that `m - n = 0` when `m = n`. So, testing for equality is as simple
as applying `sub` then applying `iszro` to the result.

{% highlight text %}
equal = λm. λn. iszro (sub m n)
{% endhighlight %}

{% highlight python %}
# equal = λm. λn. iszro (sub m n)
equal = lambda m: lambda n: iszro(sub(m)(n))

>>> equal(c3)(c2)(True)(False)
False
>>> equal(c3)(c1)(True)(False)
False
>>> equal(c3)(c3)(True)(False)
True
{% endhighlight %}

There is are two big problems with this definition, however. First, each `sub`
operation is `O(n)`, second the resulting Church numeral must be defined
otherwise `iszro` will evaluate to `tru`.

Pierce has a different definition of equal which has fewer evaluations than
mine. 
{% highlight text %}
equal = λm. λn. and (iszro (m prd n))(iszro (n prd m))
{% endhighlight %}

{% highlight python %}
# equal = λm. λn. and (iszro (m prd n))(iszro (n prd m))
equal = lambda m: lambda n: And(iszro((m)(prd)(n)))(iszro((n)(prd)(m)))

>>> equal(c2)(c3)(True)(False)
False
>>> equal(c0)(c0)(True)(False)
True
{% endhighlight %}

His definition is both more efficient and works in cases that mine does not.
Since my definition does left-to-right subtraction, negative Church numerals
(which I haven't defined) evaluate to `tru`, since `sub(2)(3) = -1`.

##Numeric Greater or Less Than

Since we can define `equal` it shouldn't be too hard to define greater and less
than, and it turns out that it isn't.

For strictly greater-than, we exploit the fact that `prd` will return 0 when
`m >= n`.  So, if `m > 0` then it could be said that the following holds true:
`m >= n && !(n >= m)` which makes `m` strictly greater-than `n`.

For strictly less-than, a simple trick of switching the order of arguments
accomplishes the same thing: `n >= m && !(m >= n)` which means that `m` must be
strictly less than `n`.

{% highlight text %}
gt = λm. λn. and (iszro (m prd n))(not iszro(n prd m))
lt = λm. λn. and (iszro (n prd m))(not iszro(m prd n))
{% endhighlight %}
{% highlight python %}
# gt = λm. λn. and (iszro (m prd n))(not iszro(n prd m))
gt = lambda m: lambda n: And(iszro((m)(prd)(n)))(Not(iszro((n)(prd)(m))))
# lt = λm. λn. and (iszro (n prd m))(not iszro(m prd n))
lt = lambda m: lambda n: And(iszro((n)(prd)(m)))(Not(iszro((m)(prd)(n))))

>>> gt(c0)(c3)(True)(False)
False
>>> gt(c0)(c0)(True)(False)
False
>>> gt(c3)(c2)(True)(False)
True
>>> lt(c0)(c3)(True)(False)
True
>>> lt(c0)(c0)(True)(False)
False
>>> lt(c3)(c2)(True)(False)
False
{% endhighlight %}

*Greater-than-or-equal* and *less-than-or-equal* can simply be calculated by
concatenating `gt|equal` and `lt|equal`, which is trivial and I'll leave that up
to the reader.

#Lists

A list can be represented by a `reduce` or `fold` function in the λ-calculus. So
the list `[x y z]` becomes a two-argument (`c n`) function that returns `c x (c
y (c z n))`. There are several steps required to build lists detailed below.

##Representing `nil`

`nil` can be represented by the same expression as `0` and `fls`, using the
arguments `c n` we can define:
{% highlight text %}
nil = λc. λn. n
{% endhighlight %}
{% highlight python %}
# nil = λc. λn. n
nil = lambda c: lambda n: n
{% endhighlight %}

##`cons` Function

`cons` is a function that will take an argument `h` and a list `t` and returns a
folded representation of `t` with `h` prepended.
{% highlight text %}
cons = λh. λt. λc. λn . c h (t c n)
{% endhighlight %}
{% highlight python %}
# cons = λh. λt. λc. λn . c h (t c n)
cons = lambda h: lambda t: lambda c: lambda n: ((c)(h))((t)(c)(n))
{% endhighlight %}

##`isnil` Function

The `isnil` function will mimic the `iszero` function, since the definition of
`nil` is the same as `0`. However, we are running it on a list, so there is a
little more to is.
{% highlight text %}
isnil = λl. l (λh. λt. fls) tru
{% endhighlight %}
{% highlight python %}
# isnil = λl. l (λh. λt. fls) tru
isnil = lambda l: (l)(lambda h: lambda t: fls)(tru)

>>> isnil(c1)(True)(False)
False
>>> isnil(nil)(True)(False)
True
{% endhighlight %}

##`head` Function

`head` is similar to `isnil`, except that we element at the beginning of the
list instead of a Church boolean, otherwise `fls`.
{% highlight text %}
head = λl. l (λh. λt.  h) fls
{% endhighlight %}
{% highlight python %}
# head = λl. l (λh. λt.  h) fls
head = lambda l: (l)(lambda h: lambda t: h)(fls)
{% endhighlight %}

##`tail` Function

`tail` is much more difficult and employs a similar trick as the
[pred function]({% post_url 2012-01-14-church-encoding %})
did. I was unable to figure out `tail` without help from the book, so here is
Pierce's solution:
{% highlight text %}
tail = λl.
         fst (l (λx. λp. pair (snd p)(cons x (snd p)))
                (pair nil nil))
{% endhighlight %}
{% highlight python %}
# tail = λl.
#          fst (l (λx. λp. pair (snd p)(cons x (snd p)))
#                 (pair nil nil))
tail = lambda l: fst((l)(lambda x: lambda p: pair((snd(p))(cons(x)(snd(p))))(pair(nil)(nil))))
{% endhighlight %}
