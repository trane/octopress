---
layout: post
title: "Concrete CESK for Dalvik"
date: 2013-02-24 16:05
comments: true
categories: dalvik, cesk, abstract interpretation
tags: racket, dalivk
---


# Introduction

The CESK machine, developed by [Matthias Felleisen][], provides a simple and
powerful architecture used to model the semantics of functional, object-oriented
and imperative languages and features like mutation, recursion, exceptions,
continuations, garbage collection and multi-threading. It is a state-machine
that takes its name from the four components of each state: Control,
Environment, Store, and Kontinuation.

In this article, I detail the CESK machine and provide a partial implementation
of an interpreter for Dalvik bytecode - the language of Android.

# CESK

Being a state machine, CESK has a notion of jumping or stepping from one state
to another. In terms of sets, we can think of a program ($$p$$) as a set of
these machine states ($$\Sigma$$) that exist within the set of all machine
states ($$\mathit{Prog}$$) with a partial transition function (`step`) from
state to state ($$\Sigma \rightharpoonup \Sigma$$).

Defining the state-space, $$\Sigma$$ as $$\sigma\in\Sigma = Stmt^*\times
FP\times Store\times Kont$$, which allows us to encode a state as a simple
struct:

`(struct state {stmts fp stor kont})`


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


# Running the CESK machine

Since the CESK machine is a state-machine, we have a single partial transition
function (from state to state) that is run until it is told to terminate (when
we encounter the *halt* continuation) called `step`. We only need to have an
initial state ($$\varsigma_0$$) and then iterate until we hit *halt*. So, we
need four things:

* `inject` to create the initial state, with an empty environment and store
* `step` the transition function for each type of state transition
* `lookup` a way to lookup frame pointers in the store
* `run` run the CESK state-machine

{% codeblock dalvik_cesk.rkt lang:racket %}
(struct state {stmts fp store kont})

; create the initial machine state ς0
(define (inject stmt)
  (state stmt empty-fp empty-store 'halt))

; the partial state to state transition function
(define (next state)
  ; extract the state struct's contents
  (let* ([stmts (state-stmts state)]
         [fp (state-fp state)]
         [σ (state-stor state)]
         [κ (state-kont state)]
         [current-stmt (first stmts)]
         [next-stmt (rest stmts)])
      (match current-stmt
        ...)))

; lookup a frame-pointer in the store
(define (lookup store fp val)
  (hash-ref store `(,val ,fp)))

; the algorithm for running the interpreter
(define (run state)
  (let ([step (next state)])
    (if (eq? 'halt (kont step))
        step
        (run step))))
{% endcodeblock %}

# Dalvik Byte-code Grammar

For the purposes of this article, I will be using the core grammar defined by
[Matt Might's Java CESK article][]. He defined this by looking at all of
Dalvik's bytecode and ensuring that its semantics are represented in a
straightforward way. He divided the language into two classes of terms:
*statements* and *expressions*.

<pre>
program ::= class-def ...

class-def ::= class class-name extends class-name
              { field-def ... method-def ... }

field-def ::= var field-name ;

method-def ::= def method-name($name, ..., $name) { body }

body ::= stmt ...

stmt ::= label label:
      |  skip ;
      |  goto label ;
      |  if aexp goto label ;
      |  $name := aexp | cexp ;
      |  return aexp ;
      |  aexp.field-name := aexp ;
      |  push-handler class-name label ;
      |  pop-handler ;
      |  throw aexp ;
      |  move-exception $name ;

cexp ::= new class-name
      |  invoke aexp.method-name(aexp,...,aexp)
      |  invoke super.method-name(aexp,...,aexp)

aexp ::= this
      |  true | false
      |  null
      |  void
      |  $name
      |  int
      |  atomic-op(aexp, ..., aexp)
      |  instanceof(aexp, class-name)
      |  aexp.field-name
</pre>

# Transitions: Evaluation and stepping

Remembering from earilier in the article, there are three types of
continutations: assignment, handler, and halt. Each of the components of a
Dalvik program will use those generalized definitions.

$$
\begin{array}{rcl}
\kappa \in \mathit{Kont} &::=&
  assign(\mathit{name}, \vec{\mathit{s}}, \mathit{fp}, \kappa)
  \\
  & \mathrel{|} & handle(\mathit{className},\mathit{label},\kappa)
  \\
  & \mathrel{|} & halt
\end{array}
$$


## Assignment: Atomic statements/expressions

Atomic expressions are expressions which evaluation must terminate, never cause
and exception or side effect.

Atomic statements assign an atomic value to a variable, this involves evaluation
of the statement/expression, calculating the frame address and updating the
store.

To evaluate an atomic expression, we use the atomic expression evaluator:

$$
\mathcal{A} : AtomicExp \times \mathit{FP} \times \mathit{Store}
\rightharpoonup \mathit{Value}
$$

Then evaluation is simply knowing what the frame pointer offset is to do a
lookup of the atomic value. Since we have encoded this with the name of the
expression along with the frame pointer we have the frame address encoded into
the store.
{% codeblock aexp_eval.rkt lang:racket %}
(define (atomic-eval e fp store)
  (match e
    [(? atom?) e]
    [(object ,classname ,op) (atomic-eval (lookup store fp (op classname)))]
    [else (lookup store fp e)]))

; A helper function to define atomic values
(define (atom? a)
  (match
    [(? void?) #t]
    [(? null?) #t]
    [(? boolean?) #t]
    [(? number?) #t]
    [else #f]))
{% endcodeblock %}

## nop, label, line

There are thre types of statements that cause no change in state: `nop`,
`label`, and `line`. We can define `nop` as: $$step(\mathbf{nop}: (\vec{s}, fp,
\sigma, \kappa) \to (\vec{s}, fp, \sigma, \kappa)$$ This says that when we se
a `nop`, get the next statement in the list of statements and run it.

The only difference between `nop` and `label` is that `label` has an identifier
(the label) with it $$step(\mathbf{label}\;\mathit{l}: (\vec{s}, fp, \sigma, \kappa))
\to (\vec{s}, fp, \sigma, \kappa)$$

`line` is defined the same as `label` and is a side-effect of the s-expression
generation, not part of the actual grammar.

Finally, we can add something to our transition function's `match` statement:

{% codeblock next_noplabel.rkt lang:racket %}
(define (next state)
  ...
    (match current-stmt
      ['(nop) (state next-stmt fp σ κ)]
      [`(line ,n) (state next-stmt fp σ κ)]
      [`(label ,l) (state next-stmt fp σ κ)])))
{% endcodeblock %}

`goto` is much like `nop`, except that we must do a lookup using the `label` to
find the next statement sequence. $$next(\mathbf{goto}\;\mathit{label} : \vec{s},
fp, \sigma, \kappa) = (S(label), fp, \sigma, \kappa)$$

{% codeblock next_noplabel.rkt lang:racket %}
(define (next state)
  ...
    (match current-stmt
      [`(goto ,l) (state (lookup-label l) fp σ κ)]
      ...
{% endcodeblock %}

## S function for Label Lookups
Labels are identifiers for statements and used in jumping from one statement to
another. We will need to store these labels for lookup later. So, let's define a
label map and then a mechanism to lookup labels. We define this mapping function
as $$\mathit{S} : \mathit{Label}\to \mathsf{Stmt*}$$

In code, what we are trying to do is to find the label and execute the next
statment, so we will need a label store, a way to update the store, and a way to
lookup the next statement by the label

{% codeblock label_stor.rkt lang:racket %}
; global label store
(define label-stor (make-hash))

; update the label store
(define (extend-label-stor label stmt)
  (hash-set! label-stor label stmt))

; lookup the statement from the label in the store
(define (lookup-label label)
  (hash-ref label-stor label))
{% endcodeblock %}

But, this means we also need to update the store when we see a label, so an
update to the earlier match construct is in order:

{% codeblock next_noplabel.rkt lang:racket %}
(define (next state)
  ...
    (match current-stmt
      [`(label ,l)
            (extend-label-stor l next-stmt)
            (state next-stmt fp σ κ)]
      ...
{% endcodeblock %}

## Continuations
{% codeblock apply_kont.rkt lang:racket %}
; Apply continuation
(define (apply-kont kont value store)
  (match kont
    ; if this is the end, not sure what to return
    ['(halt) '()]
    ; otherwise, we need to get our new state
    [`(,f ,stmts ,fp ,kaddr)
      (let ([store* (extend* store fp (lookup store fp value))])
          (state stmts store* fp kaddr))]))
{% endcodeblock %}

## Generalized Instruction

Set Dalvik has

[Matthias Felleisen]: http://www.ccs.neu.edu/home/matthias/papers.html
[Matt Might's Java CESK article]: http://matt.might.net/articles/oo-cesk/
