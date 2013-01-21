---
layout: post
title: "On Our Way to Reduction: Implement Substitution"
categories: [lambda calculus, substitution, set theory]
tags: [lambda calculus, racket]
---

#Introduction
Now that I have the
[mathematical definition of
substitution]({% post_url 2012-02-05-on-our-way-to-reduction-substitution %}),
I can implement it. For ease of implementation, I will not consider *call by
name* substitution which requires alpha-conversion. I will implement *call by
value*, which is what most familiar programming languages use, such as Python,
Ruby, PHP, C, Perl, and Java to name a few.

#Substitution: Recursive Definition with Alpha Conversion
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

#Strategy
I basically have four things to consider when substituting in *call by value*:
* If the term is in the form `[x ↦ s]x`, return the *value* `s`
* If the term is in the form `[x ↦ s]y`, return the *term* `y`, since `x ≠ y`
* If the term is in the form `[x ↦ s](λy.t1)`, return the substituted *abstraction* `λy.[x ↦ s]t1`
* If the term is in the form `[x ↦ s](t1 t2)`, return the application where each term is substituted `([x ↦ s]t1)([x ↦ s]t2)`

There are three parts to every substitution: term, variable, and value.
So I will need them as parameters to my function; I'll call them `term var value`.
In mathematical format, you can think of it like this: `[var ↦ value]term`.

#Implementation in Racket
Using a functional language, like Racket, allows us a more powerful, terse, and
elegant solution to the substitution function. I employ the use of Racket's
`match` [which gives extremely powerful pattern
matching](http://docs.racket-lang.org/reference/match.html).

{% highlight text %}
(define (subst term var value)
  (match term
    ; [x -> s]y -> y = x ? s : y
    [(? symbol?) (if (eq? term var) value term)]
    ; [x -> s]λx.b -> λx.b
    ; [x -> s]λy.b -> λy.[x -> s]b
    [`(λ (,v) ,body) (if (eq? v var) term `(λ (,v) ,(subst body var value)))]
    ; [x -> s](t1 t2)
    [`(,f ,a) `(,(subst f var value) ,(subst a var value))]))
{% endhighlight %}

#Tests
{% highlight text %}
> (subst `(λ (x) 'y) 'y '1)
'(λ (x) (1))

> (subst `(λ (x) 'y) 'y `(λ (x) 'z))
'(λ (x) ((λ (x) 'z)))

> (subst `((λ (x) y) (λ (y) z)) 'z '2)
'((λ (y) 2))

> (subst `((λ (x) y) (λ (y) z)) 'y '2)
'((λ (y) z))
{% endhighlight %}
