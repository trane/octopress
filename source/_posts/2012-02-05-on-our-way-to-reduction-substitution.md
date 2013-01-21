---
layout: post
title: "On Our Way to Reduction: Substitution"
categories: [lambda calculus, reduction]
tags: [lambda calculus, proof, substitution, reduction]
---

##Introduction
The ability to properly substitute is vital to reduction. In this post, I will
show proper and improper definitions of substitution.

##Semantics Primer
From my earlier post on
[currying]({% post_url 2012-01-14-multi-arg-handling-with-currying %})
you would have seen a substitution syntax like this, `(λx.t1)t2 ↦ [x ↦ t2]t1`
where `[x ↦ t2]t1` means the term obtained by replacing all free occurrences of
`x` in `t1` by `t2`. See my
[last post]({% post_url 2012-02-04-on-our-way-to-reduction-free-variables %})
for the definition of free variables.

##Some Wrong Ways to Do It
The
[definition of the λ-calculus is
simple]({% post_url 2012-02-04-on-our-way-to-reduction-free-variables %}).
We have a term broken into three additional parts: variable, abstraction, and
application. To define substitution, we must define it for the each part of a
term. For variables, it must be defined when there is a free variable and when
there is not.
###Naive Recursive Definition
{% highlight text %}
# Replace all free occurrences of x in x with s
[x ↦ s]x        = s
# Replace all free occurrences of x in y with s
[x ↦ s]y        = y     if x ≠ y
# Replace all free occurrences of x in λy.t1 with s
[x ↦ s](λy.t1)  = λy.[x ↦ s]t1
# Replace all free occurrences of x in t1 t2 with s
[x ↦ s](t1 t2)  = ([x ↦ s]t1)([x ↦ s]t2)
{% endhighlight %}
The problem with the definition from above is that we are relying on the *name*
of the variable or term to define substitution. This works fine if we are
careful with our choice of bound variable names, however take the example `[x ↦
y](λx.x)`. If we replace all free `x` terms within the abstraction `λx.x` with
`y` we would end up with the following: `λx.y`, clearly this is *no longer* the
identity function.

What should be taken away from this? *The names of bound variables do not
matter*

###Almost Fixed Definition
Let's ensure that names of bound variables don't matter.
{% highlight text %}
# Replace all free occurrences of x in x with s
[x ↦ s]x        = s
# Replace all free occurrences of x in y with s
[x ↦ s]y        = y     if x ≠ y
# Replace all free occurrences of x in λy.t1 with s
[x ↦ s](λy.t1)  = λy.t1         if y = x
[x ↦ s](λy.t1)  = λy.[x ↦ s]t1  if y ≠ x
# Replace all free occurrences of x in t1 t2 with s
[x ↦ s](t1 t2)  = ([x ↦ s]t1)([x ↦ s]t2)
{% endhighlight %}

So, in this case, `[x ↦ y](λx.x) = λy.y` which is indeed the identity function.
However, this is still wrong. Take this example: `[x ↦ z](λz.x)`. Using the
definition above, we would be substituting `z` for all free variables in
`(λz.x)` which would leave us with `λz.z`, the identity function. But `λz.x` is
*not* the identity function. So names are still an issue.

##Variable Capture and Capture Avoidance
When a free variable in a term `s` is bound when `s` is substituted into a term
`t` naively is called *variable capture*. We want to avoid this. We do this by
using a substitution operation that avoids mixing bound variable names of `t`
and free variable names of `s`. This is called *capture-avoiding substitution*
and is often what is implicitly meant by the term *substitution*. It is easily
achieved by one more condition on the abstraction case:
{% highlight text %}
# Replace all free occurrences of x in x with s
[x ↦ s]x        = s
# Replace all free occurrences of x in y with s
[x ↦ s]y        = y     if x ≠ y
# Replace all free occurrences of x in λy.t1 with s
[x ↦ s](λy.t1)  = λy.t1         if y = x
[x ↦ s](λy.t1)  = λy.[x ↦ s]t1  if y ≠ x and y ∉ FV(s)
# Replace all free occurrences of x in t1 t2 with s
[x ↦ s](t1 t2)  = ([x ↦ s]t1)([x ↦ s]t2)
{% endhighlight %}

This however, is not a complete definition, since it has now changed
substitution into a *partial operation*. For example:
`[x ↦ y z](λy.xy)` should equal `λy.yzy`, but because it does not appear free in
`(y z)` it never hits one of our definitions.

##A Convention: Alpha-Conversion
In order to fix the issue of bound and free variable names, we must decide to
work with terms "up to renaming of bound variables" or what Church called
*alpha-conversion*. This is the operation of consistently renaming a bound
variable in a term. In other words, "Terms that differ only in the names of
bound variables are interchangeable in all contexts".

This actually simplifies our definition, since we can change names as soon as we
get to a place where we are trying to apply the substitution to arguments where
it is undefined. This means we can completely drop the first clause of the
abstraction section.

##Final Definition
{% highlight text %}
# Replace all free occurrences of x in x with s
[x ↦ s]x        = s
# Replace all free occurrences of x in y with s
[x ↦ s]y        = y     if x ≠ y
# Replace all free occurrences of x in λy.t1 with s
[x ↦ s](λy.t1)  = λy.[x ↦ s]t1  if y ≠ x and y ∉ FV(s)
# Replace all free occurrences of x in t1 t2 with s
[x ↦ s](t1 t2)  = ([x ↦ s]t1)([x ↦ s]t2)
{% endhighlight %}
