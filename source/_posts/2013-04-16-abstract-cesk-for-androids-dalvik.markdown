---
layout: post
title: "Abstract CESK for Android's Dalvik"
date: 2013-04-16 13:24
published: false
categories: [thesis, cesk, static analysis]
tags: [racket, cesk]
---

# Introduction

Now that I have defined the [abstract semantics][] for an abstract interpreter,
it is now time to convert the [Concrete CESK Machine][] into an Abstract CESK
Machine. This important step takes us from the domain of infinite space to
finite space. After the conversion is complete, we will be able to run all
possible (and impossible) execution paths and begin reasoning about the results
and visualizing them with a control-flow graph.

# Finite State-Space

## Addresses
As I said in my [abstract semantics][] article, the solution to an infinite
address space has a practical solution: A statement is a finite thing and in any
Dalvik program, there are a finite number of statements. So I will define
and abstract address to be the statement itself:

$$
\hat{Address} = Stmt
$$

## Values

Values must be made finite, as well. For a simple example, take an integer.
Mathematically, there are an infinite amount of integers. While there is an
upper bound on integers represented in Dalvik, that doesn't stop potential
exponential blow-up in space requirements.

Imagine a statement that returns an integer. In an abstract interpreter, we must
return all possible values as a set of values. So, now, with only one statement
returning a single set of all integers that wouldn't be so terrible. But, what
if we have 10 or 100 statements that return integers?

Instead, let's represent integers as a single *abstract* value: "integer". Now,
all statements that return an integer value only have *one* value. We can take
this one step further and say that all number values can be "number" - including
floating point numbers, integers, etc.

We must do this for all possible values in Dalvik:

$$
\begin{align*}
\hat{Values} &= \mathit{Number, Object, Boolean, Null} \\
Number &= \{\mathit{Integer, Float}\} \\
Boolean &= \{\mathit{true, false} \} \\
Object &= \{ object \} \\
Null &= \{ null \}
\end{align*}
$$

## The rest

Now that we have made addresses and values finite, the rest of the CESK machine
has now become finite.

$$
\hat{\varsigma} \in \hat{\Sigma} = \vec{Stmt}\times\hat{FP}\times\hat{Store}\times\hat{Kont}\\
fp = \hat{\rho}\in\hat{Env} = \text{Var} \rightharpoonup \hat{Address}\\
\hat{\sigma} \in \hat{Store} = \hat{Address} \rightharpoonup \hat{Value}\\
\hat{a} \in \hat{Address} \text{ is a finite set of addresses} \\
\hat{Value} = \mathit{Number, Object, Boolean, Null}
$$

# The Abstract Store

The first modification to make is how we update the store. Since values in the
store are now a set, we need a way to ensure that the machine follows that
constraint.

The old store extending code looked like this:

{% codeblock lang:racket %}
; extend store with one value
(define (extend σ fp var val)
  (hash-set! σ `(,fp ,var) val))
{% endcodeblock %}

The updated code will ensure that only sets are stored as values in the store.
The `extend` function will now convert any value thrown at it into a set and
either union it into an existing set of values for that address or create a new
set for that address.

{% codeblock lang:racket %}
; extend store with a set of values
(define (extend σ addr val)
  (let ([val (cond [(set? val) val]
                   [(list? val) (list->set val)]
                   [else (set val)])])
    (hash-set σ addr (set-union
                        (hash-ref σ addr (set))
                        val))))
{% endcodeblock %}

Notice that the store is now looked up by address instead of framepointer and
variable pairs.

Likewise, the lookup function needs to be converted to use the new definition of
address:
{% codeblock lang:racket %}
; lookup a set of values for this address
(define (lookup σ addr)
    (hash-ref σ addr))
{% endcodeblock %}

# Abstract Values

We already have a function to evaluate atomic expressions, all we need is a
helper function to return the abstract atomic values `abstractify` and to have
`eval/atomic` call `abstractify`.

{% codeblock lang:racket %}
; get the set of atomic abstract values
(define (eval/atomic aexp ptr σ)
  (match aexp
    [(? atom?) (abstractify aexp)]
    [else (lookup σ ptr (abstractify aexp))]))

; get abstract atomic values
(define (abstractify aexp)
  (match aexp
    [(? void?) 'NULL]
    [(? null?) 'NULL]
    [(? boolean?) 'BOOLEAN]
    [(? number?) 'NUMBER]))
{% endcodeblock %}

This handles all value types but *Object*.


[abstract semantics]: {% post_url 2013-04-06-concrete-to-abstract-semantics %}
[Concrete CESK Machine]: {% post_url 2013-03-16-concrete-cesk-for-dalvik %}
