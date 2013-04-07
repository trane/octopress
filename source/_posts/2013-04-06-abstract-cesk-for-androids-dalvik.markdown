---
layout: post
title: "Abstract CESK for Android's Dalvik"
date: 2013-04-06 17:36
published: false
comments: true
categories: [thesis, dalvik, cesk, static analysis, android]
tags: [racket, davlik]
---

# Introduction

This is the crucial step: abstracting the concrete semantics away so that we can
do control flow analysis (CFA). Fortunately, in small-step abstract
interpretation, the concrete and abstract semantics are very similar.

In this article, I show the conversion of my [Concrete CESK machine][] to an
Abstract CESK machine and discuss the transition from abstract to concrete
semantics in with help from [Olin Shivers'][] dissertation and discovery of
*k-CFA*.

<!-- more -->

# Abstract vs Concrete

There are two main changes that occur in the transition from concrete to
abstract semantics.

* Infinite state-space must become finite state space.
* The addresses in the finite state-space become pointers to a set of *all*
  possible values for that address.

# Finite State-Space

In the [Concrete CESK machine][] that I built, it would be trivial to construct
a recursive function that allocated a new address on every call. This would
eventually fill up the available memory of the machine running it - unless, of
course, you have a computer with an infinite amount of memory. Since it is safe
to say that an infinite-memory machine doesn't exist, we must abstract away the
idea of an address.

It is both mathematically and computationally possible to define an abstract
machine with a *single* address. This is due to the nature of abstract
interpretation. Since an abstract address is a pointer to a set of possible
values, then through non-determinism, you can give all possible (and impossible)
execution paths with a single address. I will take a more practical approach.

The solution to how to represent a finite number of addresses comes from a
simple observation: A statement is a finite thing and in any Dalvik program,
there are a finite number of statements. So an *abstract* address could be the
statement itself.

With that solution, the following definition of the conversion from an infinite
store and address space to a finite store and address space follows:

$$
\hat{\varsigma} \in \hat{\Sigma} =
\vec{Stmt}\times\hat{FP}\times\hat{Store}\times\hat{Kont}
$$

In a way, one can think of the
transition from the infinite state-space to the finite-state space as a simple
"Throw hats on everything" approach. That thought would be wrong, however.

The problem with that approach is that since addresses contain frame pointers
and frame pointers contain addresses, you analysis could run into an infinite
frame pointer where its set of values contains itself.

*k-CFA* shows us how to tame recursion by removing it from the state-space
through a process called "snipping the knots" - thereby removing these cyclical
dependencies from the dependency graph.

## Making Snips

The first part of of making a snip is handled already by the Concrete CESK
machine: add a store. The store is still defined in terms of an infinite
number of addresses at this point:

$$
\sigma\in\mathit{Store} = \mathit{Address}\rightharpoonup\mathit{Value}
$$

We need to take this store and then thread it through the transition relation so
that a dependency graph edge going from set $$A\rightarrow B$$ would result in:

$$
\sigma\in\mathit{Store} = \mathit{Address}\rightharpoonup\mathit{B}
$$

Thereby making the store an integral component of each state. Fortunately, I
already did this in the Concrete CESK machine.

## Trickling up Abstraction

We need two components to calculate an "optimal" small-step abstract transition
relation.

* An abstract state-space $$\hat{\Sigma}$$
* A [Galois connection][] between the abstract and concrete state-spaces
  $$\mathcal{P}(\Sigma) \overset{\gamma}{\underset{\alpha} {\leftrightarrows}}\mathcal{P}(\hat{\Sigma})$$

### Galois Connections

Galois connections are seen mostly in order theory and is a particular
correspondence between partially ordered sets. It would be way beyond the scope
of this to delve too deeply into it. However, it is important to explain the
role the correspondence plays in static program analysis.

In [my article about static analysis][], I touch a little on lattices and
partial ordering where I introduce the concept of a supremum $$\top$$ and infimum
$$\bot$$ in terms of a lattice $$(L,\sqsubseteq)$$ and their role in showing
partial ordering of variable types.

I would like to take this a bit further and define a simple Galois connection
resulting from the mapping functions $$\alpha$$ and $$\gamma$$ from the abstract
transition relation above.

Take the following code:

{% codeblock %}
var x;
if (somefunc()) {
    x = 1 / a
}
else {
    x = 1 / b
}
{% endcodeblock %}

We can clearly see that if a or b is 0, that we have caused a problem. We can
define a single abstract function $$\alpha$$ and a single aconcretization
function $$\gamma$$ as follows:

$$
\alpha(I) = \left\{
    \begin{array}{l l}
    \bot & \text{if}\; I = \emptyset\\
    \lnot 0 & \text{if}\; \{a,b\}\subset \{\mathbb{Z} \setminus \{0\}\}\\
    \top & else\\
    \end{array} \right.
\\
\gamma(m) = \left\{
    \begin{array}{l l}
    \emptyset & \text{if}\; m = \bot\\
    \{\mathbb{Z} \setminus \{0\}\} & if m = \lnot 0\\
    \mathbb{Z} & else\\
    \end{array} \right.
$$




[Concrete CESK machine]: {% post_url 2013-03-16-concrete-cesk-for-dalvik %}
[Olin Shivers']: http://www.ccs.neu.edu/home/shivers/papers/diss.pdf
[Galois connection]: http://www.math.ksu.edu/~strecker/primer.ps
[my article about static analysis]: {% post_url 2013-01-17-on-static-analysis %}
