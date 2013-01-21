---
layout: post
title: "Interpreter: CES Machine and CPS Style"
date: 2012-11-01 19:04
comments: false
published: false
categories: [cesk, cps, interpreter]
---

#Introduction

There are many ways to implement an interpreter, like an Environment or
Substitution based. Another style of interpreters can be derived
from the simple and powerful [CESK machine][]. By desugaring a language into its
[A-Normal form][], you can make the CESK machine simpler, still.

In this article, I expand upon the work from Matt Might's article, [Writing an
interpreter, CESK-style][], by showing how you can turn a CESK machine into an
even simpler CES machine by using [Continuation Passing Style][]. This is directly
from my final project for [Scripting Languages][].

The working interpreter is provided and is written in Racket for an A-Normalized
subset of Scheme (including `call-cc`).

For a thorough overview of a CESK-machine and A-Normal form, please read
Matt Might's article above.

<!--more-->


##µScheme: The Grammar

    ; *** Input language ***

    ; <prog> ::= <defs> <exp>

    ; <defs> ::= <def> <defs>
    ;         |

    ; <def> ::= (define (<var> <formals>) <exp>)

    ; <formals> ::= <var> <formals>
    ;            |

    ; <exp> ::= <integer>
    ;        |  <var>
    ;        |  #t | #f
    ;        |  (lambda (<formals>) <exp>)
    ;        |  (let ([<var> <exp>] ...) <exp>)
    ;        |  (if <exp> <exp> <exp>)
    ;        |  (set! <var> <exp>)
    ;        |  (begin <exp> ...)
    ;        |  (<prim> <exp> ...)
    ;        |  (<exp> <exp> ...)
    ;        |  (call/cc <exp>)

##µScheme: A-Normalized Grammar

First, like in Matt's article, we'll want to put our grammar in A-Normalized
form. We can do this by simply desugaring `let` and `begin`.


    ; *** Desugared language ***

    ; <prog> ::= <defs> <exp>

    ; <defs> ::= <def> <defs>
    ;         |

    ; <def> ::= (define (<var> <formals>) <exp>)

    ; <formals> ::= <var> <formals>
    ;            |

    ; <exp> ::= <integer>
    ;        |  <var>
    ;        |  #t | #f
    ;        |  (lambda (<formals>) <exp>)
    ;        |  (if <exp> <exp> <exp>)
    ;        |  (set! <var> <exp>)
    ;        |  (call/cc <exp>)
    ;        |  (<exp> <exp> ...)
    ;        |  (<prim> <exp> ...)

#CESK to CES

To transform a grammar used by a CESK machine into one used by CES, there are a
few transformations that need to occur. First, you need to put your language
into A-Normal form.


[Scripting Languages]: http://matt.might.net/teaching/scripting-languages/spring-2012
[Writing an interpreter, CESK-style]: http://matt.might.net/articles/cesk-machines/
[CESK machine]: http://matt.might.net/articles/cesk-machines/
[A-Normal form]: http://matt.might.net/articles/a-normalization/
[Continuation Passing Style]: https://en.wikipedia.org/wiki/Continuation-passing_style
