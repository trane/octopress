---
layout: post
title: "Implementing a more powerful RegEx"
date: 2013-01-17 20:13
comments: true
published: false
categories: [regular expression]
tags: [python, javascript, ruby, regex]
---

#Introduction
In class today, I sat through my second lecture on the power behind Regular
Expressions with derivatives. The professor live coded a Regular Expression
engine during class that has all of the regular language functionality and more than the one
you'd find in perl, ruby, python, java, boost, etc. Why would language
implementers not use a better way? Because long ago it was
thought that Brzozowski's derivative method was too costly so everyone used
Thompson's method which is fast for most operations but suffers from possible
exponential blowup with some operations.

With derivatives we get those operations back (Intersection, Difference,
Compliment) and significantly decrease the complexity of implementing regular
expressions. Keep in mind, none of these are optimized and there are some types
of regular expressions that will cause exponential time (but are easily fixed
with some modifications).

I will implement all operations for regular languages in Python, Javascript and
Ruby for fun and posterity.

<!-- more -->

#Algorithm
The algorithm is a simple two step process:

* Take the derivative with respect to each character you are matching in order
* Does the final language accept $$\epsilon$$?

##A derivative of a language
In formal theory, a language is a set of characters, what we'd call a set of
strings like $$\{\text{foo},\text{bar},\text{baz}\}$$. The derivative of that
language in terms of the character *b* is
$$D_b=\{\text{foo},\text{bar},\text{baz}\}=\{\text{ar},\text{az}\}$$.

##Nullability
A nullable language is one that no longer accepts any input, in other words does
the language not accept $$\epsilon$$? Such a derivative of a language
looks like $$D_z\{\text{foo},\text{bar}\}$$, where *z* is not in the language
and thus is null.

#Operations of Regular Languages

##Matches
Here we, will define our base `RegEx` object. A character matches if the
derivative of the language with respect to that character exists.
{% codeblock regex.py lang:python %}
class RegEx:
    def isNullable(self): raise Exception()
    def derive(self, c): raise Exception()
    def matches(self, string):
        if len(string) == 0:
            return self.isNullable()
        else:
            return self.derive(string[0]).matches(string[1:])
{% endcodeblock %}
{% codeblock regex.js lang:javascript %}
var RegEx = function() {
  this.derive = function(c) {};
  this.isNullable = function() {};
  this.matches = function(s) {
    if (s.length === 0) {
      return this.isNullable();
    } else {
      return this.derive(s[0]).matches(s.substr(1));
    }
  };
  return this;
};
{% endcodeblock %}
{% codeblock regex.rb lang:ruby %}
{% endcodeblock %}

##Emptiness

$$\delta(L) =\{ \epsilon \}$$ if $$\epsilon\in L$$

$$\delta(L) = \emptyset $$ if $$\epsilon\not\in L$$

{% codeblock emptiness lang:python %}
{% endcodeblock %}
{% codeblock emptiness lang:javascript %}
{% endcodeblock %}
{% codeblock emptiness lang:ruby %}
{% endcodeblock %}
