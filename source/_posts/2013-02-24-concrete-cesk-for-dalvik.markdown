---
layout: post
title: "Concrete CESK for Dalvik"
date: 2013-02-24 16:05
comments: true
categories: dalvik, cesk
---


# Introduction

The CESK machine, developed by [Matthias Felleisen][], provides a simple and
powerful architecture used to model the semantics of functional, object-oriented
and imperative languages and features like mutation, recursion, exceptions,
continuations, garbage collection and multi-threading. It is a state-machine
that takes its name from the four components of each state: Control,
Environment, Store, and Kontinuation.

In this article, I detail the CESK machine and provide a partial implementation
for Dalvik bytecode - the language of Android.

# CESK

Being a state machine, CESK has a notion of jumping or stepping from one state
to another. In terms of sets, we can think of a program ($$p$$) as a set of
these machine states ($$\Sigma$$) that exist within the set of all machine
states ($$\mathit{Prog}$$) with a partial transition function (`step`) from
state to state ($$\Sigma \rightharpoonup \Sigma$$).

Defining the state-space, $$\Sigma$$ as $$\sigma\in\Sigma = Stmt^*\times
FP\times Store\times Kont$$, which allows us to encode a state as a simple
struct:

`(struct state {stmts fp store kont})`


## C.ontrol: A sequence of statements

The Control component of a CESK machine is a control string. In the lambda
calculus, the control string would be an expression. For Dalvik, we should think
of this component as a sequence of statements. This sequence of statements gives
an indication of which part of the program this state is.

## E.nvironment: Frame pointers

The Environment component of a CESK machine is a datastructure that maps
variables with an address. In the Dalvik CESK machine, we use simple frame
pointers as the addresses.

$$\rho\in\mathit{Env} = \mathit{Var}\rightharpoonup\mathit{Address}$$

### Addresses

Registers are offsets from frame pointers and map to local variables, we will
need to compute the location (frame offset) by pairing the frame pointer with
the name of a Dalvik register

$$FrameAddr = FramePointer\times Name\\ frameOffset : FramePointer\times Name
\to FrameAddr\\ frameOffset(fp, registerName) = (fp, registerName)$$

Objects and their fields are structurally equivalent to frame addresses.

$$FieldAddr = ObjectPointer\times Name\\ fieldOffset : ObjectPointer\times Name
\to FieldAddr\\ fieldOffset(op, fieldName) = (op, fieldName)$$

So the set of all addresses include both Object and Frame addresses:

$$a\in\mathit{Addr} = \mathit{FrameAddr}+\mathit{FieldAddr}$$

## S.tore: The Heap

The Store component of a CESK machine is a data structure that maps addresses to
values. In the Dalvik case, we map frame pointers to values.

$$ \sigma\in\mathit{Store} = \mathit{Address}\rightharpoonup\mathit{Value}\\
v\in\mathit{Value} = \mathit{Object} + \mathit{Boolean} + \mathit{Integer} +
\mathit{Null} $$

## K.ontinuations: The program stack (continuations)

The Kontinuation component of a CESK machine is essentially a program stack.
Within Dalvik, you find exception handlers and procedure calls - and with all
continuation based machines, the halt continuation to signify program
termination.

Each continuation is placed on a stack, where the top-most matching continuation
is found and executed: for exceptions this is the matching handler and for
assignment it is the next assignment continuation.

Halt is handled as a termination continuation without context encoded into the
component.

In the case of exceptions, Dalvik defines a type of exception (class name), the
branch label where execution should go and the next continuation.  $$
handle(className, label, \kappa) $$

Any other type of invocation affecting the program stack is an assignment
continuation, where the return context for a procedure call is encoded with the
register waiting for the result (name), the statement after the call, the frame
pointer from before the call and the next continuation.  $$ assign(name,
\vec{s}, fp, \kappa)$$


# Generalized Instruction

Set Dalvik has 

[Matthias Felleisen]: http://www.ccs.neu.edu/home/matthias/papers.html
