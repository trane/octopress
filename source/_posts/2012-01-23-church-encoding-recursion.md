---
layout: post
style: text
title: "Church Encoding: Recursion, Z-Combinator and Factorial"
categories: [lambda calculus, recursion, z-combinator]
tags: [python, factorial, recursion, lambda]
---

#Introduction

Now that I have the basis for an
[enriched λ-calculus]({% post_url 2012-01-16-church-encoding-enriched-calculus %})
, I can add a very important aspect to programming: *recursion*.

#Recursion-making Combinators Primer

There are several ways to arrive at recursion, some include:
* Directly with the *call-by-name* Y-combinator `Y = λf. (λx. f (x x))(λx. f (x x))`
* Derive the Y-combinator by
  [functionals](http://matt.might.net/articles/implementation-of-recursive-fixed-point-y-combinator-in-javascript-for-memoization/) and the [U-combinator](http://www.ucombinator.org)
* Use the *call-by-value* Z-combinator: `Z = λf. (λx. f (λy. x x y))(λx. f (λy. x x y))`

In the case of a *call-by-value* language such as Python, it is useless to use
the fixed-point Y-combinator since it diverges. For this post, I will be using
the Z-combinator. Pierce does not go into the details of this intricate
structure, instead opting for understanding through example.

Here is my Python implementation of the Z-combinator:
{% highlight text %}
Z = λf. (λx. f (λy. x x y))(λx. f (λy. x x y))
{% endhighlight %}
{% highlight python %}
# Z = λf. (λx. f (λy. x x y))(λx. f (λy. x x y))
Z = lambda f: (lambda x: f(lambda y: (x)(x)(y)))(lambda x: f(lambda y: (x)(x)(y)))
{% endhighlight %}

#What The Fixed-Point Combinator Does

Let's take Pierce's example of *factorial* from earlier in his book:
{% highlight text %}
factorial = λn. if n=0 then 1 else n * factorial(n-1)
{% endhighlight %}
The use of a *fixed-point* combinator is to essentially unroll the recursive
definition of `factorial` to where it occurs, where I would rewrite the above
definition unrolled:
{% highlight text %}
if n=0 then 1
else n* (if n-1=0 then 1)
    else (n-1) * (if (n-2)=0 then 1)
        else (n-2) *...))
{% endhighlight %}
You could imagine this in Church's Pure λ-calculus:
{% highlight text %}
if realeq n c0 then c1
else time n (if realeq (prd n) c0 then c1
    else times (prd n)
                (if realeq (prd (prd n)) c0 then c1
                 else times (prd (prd n)) ...)
{% endhighlight %}
The same unrolling effect is achieved when using the Z-combinator by first
defining the recursive function `g = λf.〈body containing f〉` then `h = Z g`.

#Implementation

Now that I have the abstract idea of how to generate a recursive function using
the fixed-point combinator (Z), I can implement the λ-calculus definition in
Python for our enriched (λNB) and pure versions.
{% highlight text %}
g = λfct. λn. if realeq n c0 then c1 else (times n (fct (prd n))) ;
factorial = Z g
{% endhighlight %}
{% highlight python %}
# g = λfct. λn. if realeq n c0 then c1 else (times n (fct (prd n))) ;
g = lambda fct: lambda n: (c0) if realeq(n)(c0) else (times(n)(fct((prd)(n))))
gNB = lambda fct: lambda n: 1 if n == 0 else (n * (fct(n-1)))
# factorial = Z g
factorial = Z(g)
factorialNB = Z(gNB)

>>> factorialNB(3)
6
>>> factorialNB(5)
120
>>> factorialNB(100)
93326215443944152681699238856266700490715968264381621468592963895217599993229915608941463976156518286253697920827223758251185210916864000000000000000000000000
{% endhighlight %}
