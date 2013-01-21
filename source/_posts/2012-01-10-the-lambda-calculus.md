---
layout: post
style: text
title: The Untyped/Pure Lambda Calculus
category: "lambda calculus"
tags: [abstract syntax tree, lambda, scope, redex]
---

#Introduction

The &lambda;-calculus, in its pure form does not have constants or primitive
operators like those used in arithmetic operations (this includes numbers). The
way you compute one term with another is by applying a function to its
argument(s), and an argument is always just another function.

In this post, I focus on some of the λ-calculus syntax with coding examples in
Javascript and Python.

#Basic &lambda; Syntax

&lambda;'s syntax is very simple consisting of only three kinds of terms:
variable, abstraction, and application.
* variable `x` is a term
* abstraction `λx.t` is a term called a lambda abstraction
* application `t s` is a term, where `t,s` are terms

#Abstract Syntax

Abstract syntax refers the the robust and provable representation of syntax
known as an *Abstract Syntax Tree* (AST) -- the same structure used by compilers
and interpreters. These are similar structures to the Context Free Grammar trees
from two of my previous posts, for example:
`λx. (λy. ((x y) x))` would create the following AST:

             λx
              |
             λy
              |
            apply
            /   \
        apply    x
        /   \
       x     y

This can also be written as `λx. λy. x y x`, since *application* associates to the
left and bodies of abstractions are extended as far to the left as possible.

#Scope

Scope in the λ-calculus is fairly straight-forward. You have three main parts to
scope: *bound* variables, *binders* and *free* variables.
* a variable is *bound* when it is within the body of term `t` of an abstraction of the form `λx.t`
* `λx` is a *binder* with scope `t`
* a variable `x` is *free* if its position is *not bound* by an enclosing abstraction

##Examples

Occurrences of `x` in `xy` and `λy.xy` are *free*, whereas `λx.x` and
`λz.λx.λy.x(yz)` are *bound*. For separate occurences of `x`, there can be a mix
of bound and free states such as in `(λx.x)x` where the first `x` is bound to
`λx` and the second is free.

Here is an example of `λx. λy. x y` in Javascript:
{% highlight javascript %}
function(x) {
    return function(y) {
        return x(y);
    }
}
{% endhighlight %}
Or in Python:
{% highlight python %}
lambda x: lambda y: x(y)
{% endhighlight %}

Running this program setting `x` to the `print()` function and `y` to the string
`"hello"`:
{% highlight python %}
>>> (lambda x: lambda y: x(y))(print)("hello")
hello
{% endhighlight %}

The AST created is simple:

           λx
           |
           λy
           |
         apply
         /   \
        x     y

##Scope Changes with Parenthesis

Changing this expression slightly to `λx. (λy. x) y` changes the syntax tree and
the code.

           λx
           |
         apply
         /   \
        λy    y
        |
        x

Producing the following code in Javascript:
{% highlight javascript %}
function(x) {
    var funY = function(y) {
        return x;
    }
    return funY(y);
}
{% endhighlight %}
And Python:
{% highlight python %}
(lambda x: (lambda y: x))
{% endhighlight %}

Let's run that Python code a few times and see what prints out:
{% highlight python %}
>>> (lambda x: (lambda y: x)(4))(5)
5
>>> (lambda x: (lambda y: x)(2))(8)
8
>>> (lambda x: (lambda y: x)(123))(2)
2
{% endhighlight %}

So, for every input `x,y`, we always return `x`. That has the same behavior as
the *identity function*: `λx.x`, where the only thing it does is return its
argument. So, how does `λx. (λy. x) y` have the same behavior as `λx.x`? It is
because it is a *reducible expression* (redex) that reduces to the identity
function.

#Reducible Expressions

There are many ways to reduce λ-expressions and different languages use
different strategies. The most general purpose strategy is full beta-reduction,
where you can reduce any redex at any time. To reduce `λx. (λy. x)y` we do the
following reduction in one step:

            λx. (λy. x)y
            λx. x


There are other reduction strategies employed by various languages (some use
more than one) such as:
* Call-by-name/need: Haskell
* Call-by-reference: Perl, PHP, C++
* Call-by-value: C, Scheme, OCaml




