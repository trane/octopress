---
layout: post
title: "Concrete CESK for Android's Dalvik"
date: 2013-03-11 16:05
comments: true
published: false
categories: dalvik, cesk, android
tags: racket, dalvik
---


# Introduction

At the heart of all Android applications is Dalvik byte code. It's what
everything gets compiled to and then run on the Dalvik VM. In order to do static
program analysis for Android, you need to somehow interpret the byte code.
That's where the CESK machine shines.

The CESK machine, developed by [Matthias Felleisen][], provides a simple and
powerful architecture used to model the semantics of functional, object-oriented
and imperative languages and features like mutation, recursion, exceptions,
continuations, garbage collection and multi-threading. It is a state-machine
that takes its name from the four components of each state: Control,
Environment, Store, and Kontinuation.

In this article, I implement a concrete CESK machine to interpret a dynamically
typed object-oriented language abstracted from Dalvik byte code. Every byte code
and its semantics have been transformed into this language.

# CESK

Being a state machine, CESK has a notion of jumping or stepping from one state
to another. In terms of sets, we can think of a program ($$p$$) as a set of
these machine states ($$\Sigma$$) that exist within the set of all machine
states ($$\mathit{Prog}$$) with a partial transition function (`step`) from
state to state ($$\Sigma \rightharpoonup \Sigma$$).

Defining the state-space, $$\Sigma$$ as $$\sigma\in\Sigma = Stmt^{*}\times
FP\times Store\times Kont$$, which allows us to encode a state as a simple
struct:

```(struct state {stmts fp stor kont})```

If you are familiar with pushdown automata, then a CESK machine has many
striking similarities. You can think of each state as the CES portion and the
$$\Delta\kappa$$ would be what would be pushed/popped from the stack between
state transitions. (For a project in my computational theory course some
classmates and I implemented a Non-Deterministic Pushdown Automata in Python
that outputs to a commandline and DOT format, feel free to play around with it:
[PyDA][])


## C.ontrol: A sequence of statements

The Control component of a CESK machine is a control string. In the lambda
calculus, the control string would be an expression. For Dalvik, we should think
of this component as a sequence of statements. This sequence of statements gives
an indication of which part of the program this state is.

## E.nvironment: Frame pointers

The Environment component of a CESK machine is a datastructure that maps
variables with an address. In the Dalvik CESK machine, we use simple frame
pointers as the addresses.

$$fp = \rho\in\mathit{Env} = \mathit{Var}\rightharpoonup\mathit{Address}$$

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
branch label where execution should go and the next continuation. $$
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

; lookup the value of the framepointer, variable
(define (lookup σ fp var)
  (hash-ref σ `(,fp ,var)))

; extend store with one value
(define (extend σ fp var val)
  (hash-set! σ `(,fp ,var) val))

; the algorithm for running the interpreter
(define (run state)
  (let ([step (next state)])
    (if (eq? 'halt (state-kont step))
        step
        (run step))))
{% endcodeblock %}

# Dalvik Byte-code Grammar

For the purposes of this article, I will be using the core grammar defined by
[Matt Might's Java CESK article][]. He defined this by looking at all of
Dalvik's byte code and ensuring that its semantics are represented in a
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

## Continuations

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

We can define an *applyKont* function to aid in the overall machine design,
which will be utilized when we encounter returns and exceptions.

$$
applyKont : Kont \times Value \times Store
$$

Then using applyKont with the assign and handle continuations is defined by:

$$
applyKont(\mathbf{assign}(name, \vec{s}, fp, \kappa), val, \sigma) =
  (\vec{s}, fp, \sigma[(fp, name) \mapsto val], \kappa)
  \\
applyKont(\mathbf{handle}(className, label, \kappa), val, \sigma) =
  applyKont(\kappa, val, \sigma)
$$

With this definition, we can translate this to code with `apply/κ`:

{% codeblock apply_kont.rkt lang:racket %}
(define (apply/κ κ val σ)
  (match κ
    ; assignment continuation
    [`(,name ,next ,fp ,κ_)
        (let ([σ_ (extend σ fp name val)])
          (next fp σ_ κ_))]
    ; handle continuation
    [`(,classname ,label ,κ_) (apply/κ κ_ val σ)]
    ; the termination continuation
    ['(halt) ...]))
{% endcodeblock %}

## Assignment: Atomic statements/expressions

Atomic expressions are expressions which evaluation must terminate, never cause
and exception or side effect.

Atomic statements assign an atomic value to a variable, this involves evaluation
of the statement/expression, calculating the frame address and updating the
store.

### The Atomic Expression Evaluator
To evaluate an atomic expression, we use the atomic expression evaluator:

$$
\mathcal{A} : AtomicExp \times \mathit{FP} \times \mathit{Store}
\rightharpoonup \mathit{Value}
$$

We have some key types of atomic expressions and how they are evaluated.

Atomic values that can be immediately returned such as integers, booleans, void,
null.

Register lookups simply involve knowing what the frame pointer offset is
to do a lookup of the atomic value. Since we have encoded this with the name of
the expression along with the frame pointer we have the frame address encoded
into the store with `(fp, name)`. There are two special registers `"$this"` and
`$ex`, but use the same semantics as other register lookups.

Accessing an object field is similar to register lookups, you get the field
offset from the object pointer with `(op, field)` from the store.

{% codeblock aexp_eval.rkt lang:racket %}
; AExp X FP X Store -> Value
(define (eval/atomic aexp ptr σ)
  (match aexp
    [(? atom?) aexp]
    [`(object ,name ,field)
        (lookup σ (lookup σ ptr name) field)]
    [else (lookup σ ptr aexp)]))

; A helper to determine if these are plain values
(define (atom? aexp)
  (match aexp
    [(? void?) #t]
    [(? null?) #t]
    [(? boolean?) #t]
    [(? integer?) #t]
    [else #f]))
{% endcodeblock %}

### The Atomic Assignment Statement

Like I said earlier, we need to evaluate the atomic expression and assign it a
variable-value pair in the store. We can define this operation as:

$$
next(varname := e : \vec{s}, fp, \sigma, \kappa) = (\vec{s}, fp, \sigma', \kappa)
$$

The $$\sigma'$$ is the store updated with the new atomic assignment
variable and value mapping:

$$
\sigma' = \sigma[(fp, varname) \mapsto \mathcal{A}(e, fp, \sigma)]
$$

I have a couple of articles regarding [variable substitution][] and
[implementation][] if you are interested in getting a larger view of what is
going on here.

{% codeblock atomic_assignment.rkt lang:racket %}
(define (next state)
  ...
    (match current-stmt
      [`(,varname ,aexp)
            (let* ([val (atomic-eval aexp fp σ)]
                   [σ_ (extend σ fp varname val)])
                (state next-stmt fp σ_ κ))]
      ...
{% endcodeblock %}

This creates a new state, where the store is now updated with a mapping of the
variable `var` to the value `val`.

## Object Assignment and Creation

There is another type of assignment similar to the atomic assignment statement,
a new object. This is when a variable is being assigned to a brand new object,
e.g. `Object o = new Object()`. Consequently, the definition of object
creation and assignment is similar to atomic assignments:

$$
next(varname := \mathbf{new} className : \vec{s}, fp, \sigma, \kappa) = (\vec{s}, fp, \sigma', \kappa)
$$

The $$\sigma'$$ is the store updated with the new object assignment variable and
value mapping (corresponding to a never-before used object pointer *op'*):

$$
\sigma' = \sigma[(fp, varname) \mapsto (className, \sigma')]
$$

{% codeblock object_assignment.rkt lang:racket %}
(define (next state)
  ...
    (match current-stmt
      [`(,varname new ,classname)
            (let* ([op_ `(object ,classname ,(gensym))]
                   [σ_ (extend σ fp varname op_)])
                (state next-stmt fp σ_ κ))]
      ...
{% endcodeblock %}

This creates a new state, where the store is now updated with a mapping of the
variable `varname` to the value `(object classname (gensym))`. The `(gensym)` is
used to generate a *guaranteed-to-be-globally* unique value.

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

We can add a few new items to our transition function's `match` statement:

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

{% codeblock label.rkt lang:racket %}
(define (next state)
  ...
    (match current-stmt
      [`(label ,l)
            (extend-label-stor l next-stmt)
            (state next-stmt fp σ κ)]
      ...
{% endcodeblock %}

## if-goto Statement
The `if-goto` statement is similar to a jump, the only difference being that the
conditional statement must be evaluated before you can determine which branch to
execute. We will use the `atomic-eval` that was constructed earlier to determine
the *truthiness* of the expression, then either issue a `goto` or just move to
the next statement.

$$
\mathit{next}(
\mathbf{if}\;e\;\mathbf{goto}\;\mathit{label} : \vec{s},
\mathit{fp},
\sigma,
\kappa
) =
\begin{cases}
 (\mathcal{S}(\mathit{label}), \mathit{fp}, \sigma, \kappa) & \mathcal{A}(e, \mathit{fp}, \sigma) = \mathit{true} \\
 (\vec{s}, \mathit{fp}, \sigma, \kappa) & \mathcal{A}(e, \mathit{fp}, \sigma) = \mathit{false}
\end{cases}\\
$$

{% codeblock ifgoto.rkt lang:racket %}
(define (next state)
  ...
    (match current-stmt
      ; next(if e goto label) =
      ;  if true then label
      ;  else next-stmt
      [`(if ,e goto ,l)
        ; if true, goto label, otherwise next statment
        (if (atomic-eval e fp σ)
              (state (lookup-label l) fp σ κ)
              (state next-stmt fp σ κ))]
      ...
{% endcodeblock %}

## Invoking Methods

Dalvik supports inheritance, due to a possible traversal of super classes,
method invocation is necessarily the most complicated to model. Methods involve
using all four components of the CESK machine: Control, Environment, Store,
Continuation.

It is useful to abstract away a simplified situation: assume that the method has
already been looked up. Thus, we can define an *applyMethod* helper function to
aid in applying the method to its arguments: $$applyMethod : Method \times Name
\times Value \times AExp^{*} \times FP \times Store \times Kont \rightharpoonup
\Sigma$$

Further assume a method is defined as `m = def methodName($v_1,...,$v_n) {body}`

### applyMethod Helper

*applyMethod* needs to do the following:

* Lookup the values of the arguments
* Bind those values to the formal parameters of the method
* Create a new frame pointer
* Create a new continuation
* Ensure the next sequence of statements is included in the new continuation

$$
applyMethod(m, name, val_{this}, \vec{e}, fp, σ, κ, \vec{s}) = (body, fp', σ'', κ')) \\
    fp' = \text{new} fp \\
    σ' = σ[(fp', $this) \mapsto val_{this}] \\
    σ'' = σ'[(fp', v_i) \mapsto \mathcal{A}(e_i, fp, σ)] \\
    κ' = \mathbf{assign}(name, \vec{s}, fp, κ)
$$

{% codeblock apply_method.rkt lang:racket %}
(define (apply/method m name val exps fp σ κ s)
  (let* ([fp_ (gensym fp)]
         [σ_ (extend σ fp_ "$this" val)]
         [κ_ `(,name ,s ,fp ,κ)])
    (match m
      [`(def ,mname ,vars ,body)
        (map (lambda (v e)
               (extend σ_ fp_ v (atomic-eval e fp σ)))
             vars exps)])
    `(body ,fp_ ,σ_ ,κ_)))
{% endcodeblock %}

### Invoking a method

With `apply/method` now doing much of the heavy lifting, invoking a method is
reduced to a simple method lookup. *lookup* is a partial function that traverses
through the inheritance chain until it finds the matching method. First, let's
define *invoke*, we'll need the *methodName* for our *lookup* function.

$$
next($name := \mathbf{invoke}\;\mathit{e}\;\mathit{methodName}(e_1,...,e_n),fp,\sigma,\kappa) =
\\
applyMethod(m, name, val_{this},\langle e_1,...e_n\rangle,fp,\sigma,\kappa)
$$

In order to run the *applyMethod* function, we need a few variables defined:
*val* and *m*. We can get *val* by a store lookup:

$$
val_{this} = \sigma(fp,\$this)
$$

Getting *m* is where we finally need to define *lookup* since it is what finds
the correct method. We need two values to process our lookup: *className* and
*methodName*. We already have *methodName* from our *invoke* function, and we
can get our *className* by extracting it from *val*:

$$
(op,className) = val_{this}
$$

Now that we have both *className* and *methodName* we can define our partial
function *lookupMethod* that will traverse the class hierarchy to find the correct
method to invoke:

$$
m = lookupMethod(className, methodName)
$$

In code, this sequence of functions becomes:

{% codeblock match_invoke.rkt lang:racket %}
(define (next state)
  ...
    (match current-stmt
      [`(invoke ,e ,mname ,vars)
            (let* ([val (lookup σ fp "$this")]
                   [cname (match val
                            [`(,op ,cname) cname])]
                   [m (lookup/method cname mname)])
              (apply/method m cname val vars fp σ κ next-stmt))]
      ...
{% endcodeblock %}

We can't implement `lookup/method` quite yet, however, since I still haven't
defined and implemented classes. But, for now, let's move on.

## Return Statement

A *return* is an application of the current continuation to the return value. We
already have the `apply/κ` function for application, so we just need to define
*what* to pass in the *val* parameter. In the case of a return value, we need to
evaluate the value to ensure we get the atomic instance:

$$
next(\mathbf{return}\; e,fp,\sigma,\kappa) = applyKont(\kappa, \mathcal{A}(e,
fp, \sigma), \sigma)
$$

In code, this is simply:

{% codeblock match_return.rkt lang:racket %}
(define (next state)
  ...
    (match current-stmt
      [`(return ,e) (apply/κ κ (atomic-eval e fp σ) σ)]
      ...
{% endcodeblock %}

## Exceptions

To handle exceptions we have several cases to implement: when a continuation is
an exception handler, pushing and popping exception handlers, throwing exception
handlers, and capturing exceptions.

### Exception Handler Continuation

This is the simplest continuation to handle, we simply skip over the current
continuation and we have already implemented it in the `apply/κ` function, so
there is no need to do anything more. This would happen if the current
continuation was an exception, but no exception was needed.

### Pushing and Popping Exception Handlers

We will have two ways to put and get exception handlers, but pushing and popping
from the program stack with a *pushhandler* and a *pophandler*:

$$
next(\mathbf{pushhandler}\;className\;label\; : \vec{s},fp,\sigma,\kappa) =\\
(\vec{s}, fp, \sigma, \mathbf{handle(className, label, \kappa)}) \\
next(\mathbf{pophandler}:\vec{s},fp,\sigma,\mathbf{handle}(className, label,
\kappa)) = (\vec{s},fp,\sigma,\kappa) $$

In code:
{% codeblock match_pushpop.rkt lang:racket %}
(define (next state)
  ...
    (match current-stmt
      [`(pushhandler ,name ,l)
        `(,next-stmt ,fp ,σ ,(cons `(,name ,l ,κ) κ))]
      [`(pophandler)
        `(,next-stmt ,fp ,σ ,(car κ))]
      ...
{% endcodeblock %}

### Throwing Exception Handlers

In order to throw an exception, we must search the stack for a matching
exception handler. Implementing a helper function *handle* to do this for us
will help. First, let's define *helper* as:

$$
handle : Value \times FramePointer \times Store \times Kont \rightharpoonup
\Sigma
$$

Here is how we will traverse the stack, putting last thrown exceptions into the
register *"$ex"* as is protocol:

If *className* is a subclass of *className'*:

$$
handle((op,className),fp,\sigma,\mathbf{handle}(className', label, \kappa)) =\\
(\mathcal{S}(label), fp, \sigma[(fp,\$ex) \mapsto (op, className)], \kappa)
$$

If not:

$$
handle((op,className),fp,\sigma,\mathbf{handle}(className', label, \kappa)) =\\
handle((op, className),fp,\sigma,\kappa)
$$

A *throw* skips over non-handler continuations:

$$
handle(val,fp,\sigma,\mathbf{assign}(name,\vec{s},fp',\kappa)) =
handle(val,fp',\sigma,\kappa)
$$

*handle* in code looks like this:

{% codeblock handle.rkt lang:racket %}
(define (handle o fp σ ex)
  (match `(,o ,ex)
    ; handler continuation
    [`((,op ,name) (,name_ ,l ,κ))
        (if (isinstanceof name_ name)
            `(,(lookup-label l) ,fp ,(extend σ fp "$ex" o) ,κ)
            (handle o fp σ κ))]
    ; non-handler continuation
    [`(,val (,name ,s_ ,fp_ ,κ)) (handle val fp_ σ κ)]))
{% endcodeblock %}

You might have noticed a call to `isinstanceof`. I will define this function
later when defining classes.

With the *handle* helper function, we can now define the throw statement:

$$
next(\mathbf{throw}\;e : \vec{s},fp,\sigma,\kappa) =
handle(\mathcal{A}(e,fp,\sigma),fp, \sigma, \kappa)
$$

Then we can add this to the transition function:
{% codeblock match_throw.rkt lang:racket %}
(define (next state)
  ...
    (match current-stmt
      [`(throw ,e) (handle (atomic-eval e fp σ) fp σ κ)]
      ...
{% endcodeblock %}

### Capturing Exceptions

Since we store the last thrown exception into the *$ex* register, it can be
examined to determine which exception was caught. We then use this to go to the
label that handles this execution branch.

We can define this capturing with a *moveException* statement:

$$
next(\mathit{moveException}\;e : \vec{s}, fp, \sigma, \kappa) = (\vec{s}, fp, \sigma',
\kappa)
$$

The store is updated by simply moving the exception *e* into the register
*$ex*:

$$
\sigma' = \sigma[(fp, \$ex) \mapsto e]
$$

Putting this in our transition function, this is:

{% codeblock match_exception.rkt lang:racket %}
(define (next state)
  ...
    (match current-stmt
      [`(move-exception ,e)
          (let ([$ex (lookup σ fp "$ex")])
            `(,next-stmt ,fp ,(extend σ fp $ex e)))]
      ...
{% endcodeblock %}

## Class Definitions

With the calls to the unimplemented `lookup/method` and `isinstanceof`
functions, it's time to implement them! But, first we need to define and
implement classes.

Classes are defined by their name, a potential super class, 0 or more fields
(instance variables), and 0 or more methods.

$$
classDef : Name \times Super \times Fields \times Methods \\
fieldDef : Name \\
methodDef : Name \times Formals \times Body \\
superDef : Name \times Void
$$

What we need is to be able to represent classes in a sane way that will allow us
to keep track of super classes, fields, and methods. A *classDef* then, in code
is: `(struct class {super fields methods})` where `super` is the name of the
super class, `fields` is a list of field names, and `methods` is a mapping
defined by:

$$
class.methodName \rightharpoonup methodDef
$$

Since methods also have multiple properties, we will also define a method as:
`(struct method {formals body})`.

We will also need a way to store and lookup classes. Since class names are
unique, we can define a simple table of classes and a *lookupClass* method
defined as:

$$
classTable = ClassName \rightharpoonup ClassDef \\
class = lookupClass(className)
$$

Now in code, classes are:

{% codeblock classes.rkt lang:racket %}
(struct class {super fields methods})
(struct method {formals body})

(define class-table (make-hash))

(define (lookup/class name)
  (hash-ref class-table name))

(define (extend-class-table name class)
  (hash-set! class-table name class))
{% endcodeblock %}

### Method Lookup

Now that we have defined classes, method lookup is a recursive function that
traverses the classes hierarchy until it reaches a matching method, defined by

$$
m = lookup(className, methodName)
$$

In code:

{% codeblock lookup_method.rkt lang:racket %}
(define (lookup/method classname name)
  (let* ([class (lookup/class classname)]
         [methods (class-methods class)]
         [super (class-super class)])
    (if (hash-has-key? methods name)
        (hash-ref methods name)
        (if (void? (class-super class))
            (lookup/method (class-super class) name)
            (void)))))
{% endcodeblock %}

Since *super* is just a string, all we need to do is check for string equality.
By definition, a class without a super class is *void*.

### Is Instance Of

Finding out if a class is an instance of another class is simple: find out if
the class name is anywhere in its direct class hierarchy, returning true if it
is and false otherwise:

$$
instance = isinstanceof(superClass, baseClass)
$$

In code:

{% codeblock is_instance_of.rkt lang:racket %}
(define (isinstanceof super base)
    (if (void? super)
        #f  ; reached the top class
        (if (equal? super base)
            #t
            (let* ([class (lookup/class base)]
                   [upper (lookup/class super)])
              (if (equal? (class-super class) super)
                  #t
                  (isinstanceof (class-super upper) super))))))
{% endcodeblock %}

Since *super* is just a string, we need to only check string equality. By
definition, *super* is *void* if there is no higher class.

[Matthias Felleisen]: http://www.ccs.neu.edu/home/matthias/papers.html
[Matt Might's Java CESK article]: http://matt.might.net/articles/oo-cesk/
[implementation]: {% post_url 2012-02-15-on-our-way-to-reduction-implement-substitution %}
[variable substitution]: {% post_url 2012-02-05-on-our-way-to-reduction-substitution %}
[PyDA]: https://github.com/trane/PyDA
