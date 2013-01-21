---
layout: post
title: "Reduction"
categories: [lambda calculus, reduction]
tags: [racket, lambda calculus]
---

##Introduction
Two of the most common forms of reduction are *Call-By-Name* (CBN) and
*Call-By-Value* (CBV), with the latter being the most ubiquitous. In this post,
I give implement both in [Racket](http://racket-lang.org).

##Mathematical Definitions
### Non Deterministic Full ß-reduction
This is a *non-deterministic*, full ß-reduction definition. Notice that there is
no *value* term definition, that is because a value cannot be a ß-redex. The way
to read the following rules are, the definition on top of the line is the
conditional, on the bottom is what happens if that is true.

$$
\frac{\text{if term is redex}}{\text{then reduce and substitute}}
$$

<!--more-->
###Application
In application, there are two scenarios that need to be handled: the left hand
and/or right hand terms are ß-redexes.

$$
\frac{t_1 \rightarrow t\prime_1}{t_1 t_2 \rightarrow t\prime_1 t_2}
$$

$$
\frac{t_2 \rightarrow t\prime_2}{t_1 t_2 \rightarrow t_1 t\prime_2}
$$

###Abstraction
If the body of a λ-abstraction is a ß-redex, then reduce it and substitute.

$$
\frac{t_1 \rightarrow t\prime_1}{\lambda x.t_1 \rightarrow \lambda x.t\prime_1}
$$

###Application-Abstraction
If we have an abstraction and a term, substitute the term into the body of the
abstraction if the [free variables]({% post_url 2012-02-04-on-our-way-to-reduction-free-variables %})
in the body match the term.

$$
(\lambda x. t\_{12} ) t_2 \rightarrow \[x \rightarrow t_2] t\_{12}
$$

##ß-reduction Implementation
###Substitution
You can see the
[implementation]({% post_url 2012-02-15-on-our-way-to-reduction-implement-substitution %})
and
[theory]({% post_url 2012-02-05-on-our-way-to-reduction-substitution %})
posts on substitution to get more background. Here is the implementation in
Racket:

{% highlight scheme %}
(define (subst term var value)
  (match term
    ; [x -> s]y -> y = x ? s : y
    [(? symbol?) (if (eq? term var) value term)]
    ; [x -> s]λx.b -> λx.b
    ; [x -> s]λy.b -> λy.[x -> s]b
    [`(λ (,v) ,body) (if (eq? v var) term `(λ (,v) ,(subst body var value)))]
    ; [x -> s](t1 t2) -> [x -> s]t1 [x -> s]t2
    [`(,f ,a) `(,(subst f var value) ,(subst a var value))]))
{% endhighlight %}

###Beta-reduction
Beta-reduction is not *full* beta reduction. We only have one rule to implement,
*abstraction*

{% highlight scheme %}
; (t1 t2)
(define (ß-reduce term)
  (match term
      [`((λ (,v1) ,b1), (and rhs `(λ (,v2) ,b2))) (subst b1 v1 rhs)]))
{% endhighlight %}

###Full ß-reduction
{% highlight scheme %}
(define (full-ß-reduce term)
  (match term
      ; value
      [(? symbol?) (error "Cannot reduce value" term )]
      ; identity
      [`(λ (,v) ,v) (term)]
      ; abstraction
      [`((λ (,v1) ,b1)(λ (,v2) ,b2)) (ß-reduce term)]
      ; application
      [`((λ (,v1) ,b1), e) (full-ß-reduce e)]
      ; application-abstraction
      [`(,f ,e) (full-ß-reduce f)]))
{% endhighlight %}

##Tests
###Beta-Reduce Tests
{% highlight text %}
;(ß-reduce `((λ(x) (x x))(λ(z) u)))
;'(((λ (z) u)) ((λ (z) u)))
;(ß-reduce `((λ(x)(λ(y) (y x)))(λ(z)u)))
;'(λ (y) (y ((λ (z) u))))
{% endhighlight %}
###Full ß-reduce Tests
{% highlight text %}
;> (full-ß-reduce `((λ(y) (y a))((λ(x)x)(λ(z)((λ(u)u)z)))))
;'((λ (z) ((λ (u) u) z)))
;> (full-ß-reduce `((λ(x)(x x))(λ(x)(x x))))
;'(((λ (x) (x x))) ((λ (x) (x x))))
{% endhighlight %}
