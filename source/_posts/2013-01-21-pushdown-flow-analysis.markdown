---
layout: post
title: "CESK and Pushdown System for Analysis"
date: 2013-01-21 14:57
published: false
comments: true
categories: [static program analysis, flow analysis, pushdown analysis]
tags: [pushdown systems, abstract interpretation, higher-order languages]
---

# Introduction
One of the biggest issues with control-flow analysis is loss of precision.
Mainly, this comes from handling recursion by bounding the stack. This bound is
critical, since no one has an infinite amount of memory in their computer. With
the recent work on context-free control flow analysis [CFA2][] by Dimitrios
Vardoulakis and Olin Shivers, the ability to *precisely* analyse recursion while
just skirting undecidability has been found.

In this article I discuss how with help from CFA2, can be transformed into CESK-machine-style
pushdown system

"[Introspective Pushdown Analysis of Higher-Order Programs][]" by Christopher
Earl, Ilya Sergey, Matthew Might, and David Van Horn.


#Context Free
The type of machines used to compute Context-Free languages are Deterministic and
Non-Deterministic Pushdown Automata.


[CFA2]: http://arxiv.org/pdf/1102.3676
[Introspective Pushdown Analysis of Higher-Order Programs]: http://matt.might.net/papers/earl2012ipdcfa.pdf
