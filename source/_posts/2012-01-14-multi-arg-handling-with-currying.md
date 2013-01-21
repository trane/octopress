---
layout: post
style: text
title: Multi-Argument Handling Through Currying
categories: [lambda calculus, currying]
tags: [python, perl, javascript, lazy]
---

#Introduction

Since the λ-calculus does not have numbers or operators, only functions, it
seems limited and useless, but with simple constructs one can create numbers,
then build to operators, then data structure like lists. In fact, the λ-calculus
is Turing Complete! The λ-calculus also lacks the built-in ability to handle
multiple arguments, instead it employs Currying.

In this post I implement a simple multi-argument function transformed into a
curried form in Python, Javascript and Perl.

#Primer on Multi-Argument Functions

We must understand
that the λ-calculus has no built-in ability to handle multiple arguments for a
function - or abstraction. Instead, we rely on a transformation of multiple
arguments to higher-order functions called *currying* - named after
Haskell Curry. The principle is straight-forward: "Suppose that `s` is a term
involving two free variables `x` and `y` and that we want to write a function
`f` that, for each pair `(v,w)` of arguments, yields the result of substituting
`v` for `x` and `w` for `y` in `s`." (Pierce).

So, as an example of what you would do in a rich programming language
for a multiple argument function `f = λ(x,y).s`, where `v = 2x` and `w =
3y`:

##Rich Language Examples

Python:
{% highlight python %}
def f(x,y):
    return 2*x + 3*y
{% endhighlight %}
Javascript:
{% highlight javascript %}
var f = function(x,y) {
    return 2*x + 3*y;
}
{% endhighlight %}
Perl:
{% highlight perl %}
sub f {
    my (x,y) = @_;
    return 2*x + 3*y;
}
{% endhighlight %}

##Currying

In the λ-calculus, we employ currying where the expression `f = λ(x,y).s` is
transformed into `f = λx.λy.s`. This simply means that `f` is a function that
returns a function when given a value `v` for `x`, that returned function then
returns the result when given a value `w` for `y`. In reducible expression form:
`f v w` reduces to `((λy.[x → v]s)w)` once the value for `x` is passed, then is
reduced to `[y → w][x → v]s`.

In the following examples, I continue with `v = 2x` and `w = 3y`.

##Curried Examples

Python (built-in lambda):
{% highlight python %}
(lambda x: (lambda y: 2*x + 3*y)(3))(2)
>>>13
{% endhighlight %}
Javascript (anonymous functions):
{% highlight javascript %}
(function(x) {
    return (function(y) {
        return 2*x + 3*y;
    });
})(2)(3);
>>>13
{% endhighlight %}
Perl (anonymous subroutines):
{% highlight perl %}
sub {
    my $x = shift;
    return sub {
        return 2*x + 3*shift;
    };
}->(2)->(3);
>>>13
{% endhighlight %}
