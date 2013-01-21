---
layout: post
style: text
title: Safe Languages and Dynamic Checking
categories: [static analysis]
---

#Introduction

I recently cracked open Benjamin C. Pierce's [Types and Programming
Languages](http://www.amazon.com/gp/product/0262162091/ref=as_li_qf_sp_asin_tl?ie=UTF8&tag=blogerrstrcom-20&linkCode=as2&camp=1789&creative=9325&creativeASIN=0262162091)
in anticipation of the upcoming semester's research. I will often times skim
through introductions, but decided to carefully analyze what I was reading this
time - and I'm glad I did! For quite some time I've wondered what the purpose of
such verbose type systems, like Java has, had any purpose beyond contributing to
<del>carpal tunnel</del> a visual queue to the programmer. Coming into the
programming world through Perl and Ruby, I
learned how to infer what a variable does by the context of the statement and
fail to see the benefit of having to explicitly state it. Compilers can do type
inference, too, so what is the point of all the extra declarations?

#Statically Checked ?= Safe Language
I have heard
numerous explanations for why we must type <code>ArrayList array = new
ArrayList()</code>, the most often cited is that it makes it a safe language.
That, intuitively, is false. If that were the case, C++ would be a safe language
but any language that allows you to write past the bounds of an array into some
unknown block is categorically *unsafe*.

Well, it turns out that I was almost right: verbose type declarations are often
times there to <del>annoy</del> help the programmer, not the compiler. Explicit
type declaration allows the language to be statically checked for sanity,
however static type checking does not guarantee a safe language, nor does it
guarantee run-time safety, e.g. array bounds checking is still done dynamically.

There has been much work done in the area of static analysis for dynamically
typed languages. In fact, much of the research [Matt
Might](http://matt.might.net) does is in this area of higher-order languages.

#Dynamically Checked ?= !Safe Language
Scheme, Perl, Lisp, etc are all *safe* languages. In other words, it is
"impossible to shoot yourself in the foot while programming" (Pierce)

#What is a Safe Language?
According to Pierce, a safe language is one that "protects its own
abstractions", his examples include accessing arrays by built in methods,
lexically scoped variables only being accessible in that scope, and stacks that
behave as stacks.

Variable scope is an interesting attribute of a safe language, many languages
handle this differently. Perl, for example, uses the `my` keyword to limit the
scope of variables and enforces this when you explicitly declare `use strict`.
Take the following useless bit of code:
{% highlight perl %}
sub fun {
    my ($arrayref) = @_;
    my %hash;
    my $value = 0;
    for my $element (@$arrayref) {
        my $thing = $element;
        $hash{$element} = $thing;
    }
    # this should print 0..arrayref length
    print map { "$_\n" if $arrayref->[$_] == $hash{$_}} @$arrayref;

    print $thing # out-of-scope, error
    print $element # out-of-scope, error
}

{% endhighlight %}
However, this is completely different than scope in Javascript, since it has
function-level scoping. `var` is the equivalent to Perl's `my` keyword, since
all variables without a `var` are globally scoped.
{% highlight javascript %}
function fun(array) {
    var hash = {},
        value = 0;
    for (var i = 0, len = array.length; i < len; i++) {
        var thing = array[i];
        hash[thing] = thing;
    }
    // this will never loop, since 'i' is scoped to the function 'fun' and not
    // the for loop
    for (; i < len; i++) {
        if (array[i] === hash[array[i]]) {
            console.log(array[i]);
        }
    }
    console.log(thing); // prints last element of array
    console.log(i, len); // prints the values of these
}
{% endhighlight %}
I would argue that while Javascript is a bit strange here with function-level
scoping only, it passes the scoping test in Pierce's safe language definition,
as well as passing the array-bounds checking test and a stack that acts like a
stack. In fact, it is the safety of Javascript that allows tools like Mozilla's
[Spider Monkey](https://developer.mozilla.org/en/SpiderMonkey), Google's
[V8](http://code.google.com/p/v8/) (which is what node.js runs on) to infer type
information.

#Do We Need Statically Typed Languages?
I don't know. It seems that much of the most recent languages are all
dynamically typed. I will revisit this question at a later date, when I've made
my way through more than just the first few chapters of this book. At the
moment, it seems like an antiquated and restrictive way of analyzing programs.
