---
layout: post
style: text
title: "Church Encoding: Boolean, Numerals and Arithmetic"
categories: [lambda calculus, church encoding]
tags: [python, church numerals, church boolean]
---

#Introduction

In my [previous post]({% post_url 2012-01-14-multi-arg-handling-with-currying %}) about Currying, I mentioned that the λ-calculus has no
primitive numbers or operation, just functions and more functions. In this post,
I explore how, through simple constructs, Church was able to implement numbers,
booleans, arithmetic operations and conditionals with examples in Python.

Please see my [previous post]({% post_url 2012-01-14-multi-arg-handling-with-currying %}) on Currying, as it is critical to understanding the
material here.

#Church Booleans

There is no such thing as a "true" or "false" in the λ-calculus. However, we can
*represent* those boolean values by defining the λ terms `tru` and `fls`.

{% highlight python %}
# tru = λt.λf.t
tru = lambda t: lambda f: t
# fls = λt.λf.f
fls = lambda t: lambda f: f
{% endhighlight %}

Those terms can be used to test if a value is `true` or `false` with a term
`test b v w` which *reduces* to `v` if `b` is `tru` and `w` if `fls`.
{% highlight python %}
# test = λl.λm.λn. l m n
test = lambda l: lambda m: lambda n: (l)(m)(n)
{% endhighlight %}
To see how `test` reduces when called as `test tru v w`, we must first expand it
and then we'll use call-by-value reduction

      test tru v w
    → (λl.λm.λn. l m n) tru v w
    → (λm.λn. tru m n)v w
    → (λn. tru v n)w
    = tru v w
    → (λt.λf.t)v w
    → (λf.v)w
    = v

Making some test runs with our Python implementation:
{% highlight python %}
>>> test(tru)(True)(False)
True
>>> test(fls)(True)(False)
False
{% endhighlight %}

##Boolean Logical Operations

Since we now have the ability to represent boolean values, we can implement
boolean algebra!

Implementing a logical AND `and = λb.λc. b c fls`, we return `c` if `b` is `tru`
or `fls` if `b` is `fls`. So if `b` is `tru` and `c` is `fls`, we return `c`
(meaning `fls`), otherwise if `b` and `c` are `tru`, we return `tru`.
{% highlight python %}
# and = λb.λc. b c fls
And = lambda b: lambda c: (b)(c)(fls)

>>> And(tru)(tru)(True)(False)
True
>>> And(tru)(fls)(True)(False)
False
>>> And(fls)(tru)(True)(False)
False
{% endhighlight %}

To implement OR: `or = λb.λc. b tru c`, meaning if `b` is `fls` return `c`, if `b`
is `tru` return `tru`. This, of course, would not implement XOR, since in the
event that `b` is `tru`, we automatically return `tru`.
{% highlight python %}
# or = λb.λc. b tru c
Or = lambda b: lambda c: (b)(tru)(c)
>>> Or(fls)(fls)(True)(False)
False
>>> Or(fls)(tru)(True)(False)
True
>>> Or(tru)(tru)(True)(False)
True
>>> Or(tru)(fls)(True)(False)
True
{% endhighlight %}

To implement NOT: `not = λb. b fls tru`, meaning return the opposite of `b`. We
can apply to AND and OR operations along with boolean values.
{% highlight python %}
# not = λb. b fls tru
Not = lambda b: (b)(fls)(tru)
>>> Not(tru)(True)(False)
False
>>> Not(fls)(True)(False)
True
>>> Not(And(tru)(tru))(True)(False)
False
>>> Not(And(tru)(fls))(True)(False)
True
>>> Not(Or(fls)(tru))(True)(False)
False
>>> Not(Or(fls)(fls))(True)(False)
True
{% endhighlight %}

##Pairs

Now that we have booleans, we can encode pairs of terms into one term, getting
the first and second projections (`fst` and `snd`) when we apply the correct
boolean value:

    pair = λf.λs.λb. b f s
    fst = λp. p tru
    snd = λp. p fls

This means if we apply boolean value `b` to the function `pair v w`, it applies
`b` to `v` and `w`. The application yields `v` if `b` is `tru` and `w`
otherwise. Reducing the redex `fst(pair v w) →* v` goes as follows:

      fst(pair v w)
    = fst((λf.λs.λb. b f s)v w)
    → fst((λs.λb. b v s)w)
    → fst((λb. b v w))
    = (λp. p tru)(λb. b v w)
    → (λb. b v w)tru
    → tru v w
    = v

{% highlight python %}
# pair = λf.λs.λb. b f s
pair = lambda f: lambda s: lambda b: (b)(f)(s)
# fst = λp. p tru
fst = lambda p: (p)(tru)
# snd = λp. p fls
snd = lambda p: (p)(fls)

>>> fst(pair(tru)(fls))(True)(False)
True
>>> fst(pair(tru)(tru))(True)(False)
True
>>> fst(pair(fls)(tru))(True)(False)
False
>>> fst(pair(fls)(fls))(True)(False)
False
>>> snd(pair(fls)(fls))(True)(False)
False
>>> snd(pair(fls)(tru))(True)(False)
True
>>> snd(pair(tru)(fls))(True)(False)
False
>>> snd(pair(tru)(tru))(True)(False)
True
{% endhighlight %}

#Church Numerals

So far, we have been able to implement the basis of Boolean Algebra with *only*
functions! Church uses a slightly more intricate representation of numbers by
use of composite functions. Basically, to represent a natural number `n`, you
simply encapsulate an argument `n` times with a successor function `scc`. You
can think of `scc = n + 1`. To represent 0, Church used the same definition as
`fls` (our False representation) - this should be familiar to most programmers
as `0 == False` in many languages. So a break down of 0..Nth Church numeral is
as such:

    c0 = λs. λz. z
    c1 = λs. λz. s z
    c2 = λs. λz. s (s z)
    c3 = λs. λz. s (s (s z))
    ... and so on

The `scc` combinator is defined as: `scc = λn. λs. λz. s (n s z)`

It works by combining a numeral `n` and returns another Church numeral. It does
this by yielding a function that takes `s` and `z` as arguments and applies `s`
repeatedly to `z`, specifically `n` times.

*Note:* For the example output below, the two arguments to each Church numeral
are a lambda function that is equivalent to `scc` and the numeral `0`, since
those are the parameters required for a Church numeral. It merely maps each
function to the natural number value it represents. `c0->0, c1->1`.
{% highlight python %}
# scc = λn. λs. λz. s (n s z)
scc = lambda n: lambda s: lambda z: (s)((n)(s)(z))
c0 = lambda s: lambda z: z
c1 = scc(c0)
c2 = scc(c1)
c3 = scc(c2)

>>> (c1)(lambda n: n+1)(0)
1
>>> (c2)(lambda n: n+1)(0)
2
>>> (c3)(lambda n: n+1)(0)
3
{% endhighlight %}

*Addition* is essentially the `scc` combinator applied `m` times to a Church
numeral `n`, where `n + m = v`. Equivalently, `scc` is merely `plus` applied
once.

    plus = λm. λn. λs. λz. m s (n s z)

{% highlight python %}
# plus = λm. λn. λs. λz. m s (n s z)
plus = lambda m: lambda n: lambda s: lambda z: (m)(s)((n)(s)(z))

>>> plus(c1)(c2)(lambda n: n+1)(0)
3
>>> plus(c3)(c2)(lambda n: n+1)(0)
5
{% endhighlight %}

*Multiplication* is the repeated application of `plus`, since `2+2+2 = 2*3`.

    times = λm. λn. m (plus n) c0

{% highlight python %}
times = lambda m: lambda n: (m)(plus(n))(c0)

>>> times(c2)(c3)(lambda n: n+1)(0)
6
>>> times(c3)(c3)(lambda n: n+1)(0)
9
{% endhighlight %}

*Exponentiation* uses repeated multiplication to get the intended value since
`2**3 = 2*2*2`.

    exp = λm. λn. n (times m) c1

This translates to, `(m * 1)**n`.
{% highlight python %}
exp = lambda m: lambda n: (n)(times(m))(c1)

>>> exp(c3)(c3)(lambda n: n+1)(0)
27
>>> exp(c3)(c0)(lambda n: n+1)(0)
1
>>> exp(c0)(c1)(lambda n: n+1)(0)
0
>>> exp(c2)(c1)(lambda n: n+1)(0)
2
>>> exp(c3)(c2)(lambda n: n+1)(0)
9
{%endhighlight %}

*Subtraction* requires quite a bit more work to function, involving the
predecessor combinator `prd`. First we must define two pairs `zz` (a starting
value) and `ss` that takes two arguments `ci, cj` then yields `cj, cj+1`.
Applying `ss`, `m`times to pair `c0,c0` yields `0,0` when `m` is 0, otherwise
`cm-1, cm` when `m` is positive. The `pred` is always found in the first
component of the pair.

    zz = pair c0 c0
    ss = λp. pair (snd p) (plus c1 (snd p))
    prd = λm. fst (m ss zz)

{% highlight python %}
# zz = pair c0 c0
zz = pair(c0)(c0)
# ss = λp. pair (snd p) (plus c1 (snd p))
ss = lambda p: pair(snd(p))(plus(c1)(snd(p)))
# prd = λm. fst (m ss zz)
prd = lambda m: fst((m)(ss)(zz))
{% endhighlight %}

Now that we have the `prd` combinator, we can define subtraction. Like addition,
where we find a successor by iteratively adding 1 to 0, we can subtract by
iteratively subtracting 1, n-times from `m`, where `m - n = v`.

    sub = λm. λn. n prd m

{% highlight python %}
# sub = λm. λn. n prd m
sub = lambda m: lambda n: (n)(prd)(m)

>>> sub(c3)(c1)(lambda n: n+1)(0)
2
>>> sub(c3)(c2)(lambda n: n+1)(0)
1
>>> sub(c0)(c0)(lambda n: n+1)(0)
0
{% endhighlight %}
