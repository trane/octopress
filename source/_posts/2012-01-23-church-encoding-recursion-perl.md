---
layout: post
style: text
title: "Z-Combinator and Factorial in Perl"
categories: [lambda calculus, recursion, z-combinator]
tags: [perl, factorial, lambda calculus]
---

#Introduction

Since I just did an implementation of the
[Z-combinator and Factorial in Python]({% post_url 2012-01-23-church-encoding-recursion %}),
I figured it would be fun to implement in Perl, too. It's a lot uglier in Perl
than Python, but Python is cheating with having a built-in `lambda` operator.

#Implementation
{% highlight text %}
Z = λf. (λx. f (λy. x x y))(λx. f (λy. x x y))
g = λfct. λn. if realeq n c0 then c1 else (times n (fct (prd n))) ;
factorial = Z g
{% endhighlight %}
{% highlight perl %}
#!/usr/bin/env perl

use warnings;
use strict;

my $Z = sub {
    my ($f) = @_;
    return (sub {
        my ($x) = @_;
        return $f->(sub {
            my ($y) = @_;
            return $x->($x)->($y);
        });
    })->(sub {
        my ($x) = @_;
        return $f->(sub {
            my ($y) = @_;
            return $x->($x)->($y);
        });
    })
};

my $g = sub {
    my ($fct) = @_;
    return sub {
        my ($n) = @_;
        return !$n ? 1 : $n * $fct->($n-1);
    };
};

my $factorial = $Z->($g);

print map { $factorial->($_)."\n" } @ARGV;
{% endhighlight %}
So what is the output?
{% highlight text %}
./thelambda.pl 3 5 100
6
120
9.33262154439441e+157
{% endhighlight %}
