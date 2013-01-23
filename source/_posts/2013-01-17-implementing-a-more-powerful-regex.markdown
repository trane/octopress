---
layout: post
title: "Implementing a more powerful Regular Expression Engine in 3 Languages"
date: 2013-01-22 20:13
comments: true
published: true
categories: [regular expressions]
tags: [python, javascript, ruby, regex]
---

#Introduction
In class today, I sat through my second lecture on the power behind Regular
Expressions with derivatives. The professor live coded a Regular Expression
engine in Python during class that has all of the regular language functionality
and more than the ones you will find in perl, ruby, python, java, boost, etc.

Why would language
implementers not use the more powerful way? Because long ago it was
thought that Brzozowski's derivative method was too costly so everyone used
Thompson's method which is fast for most operations but suffers from possible
exponential blowup with some operations.

With derivatives we get those operations back (Intersection, Difference,
Complement) and significantly decrease the complexity of implementing regular
expressions.

I will implement all operations for regular languages in Python, Javascript and
Ruby for fun and posterity.

<!-- more -->

#Warning

Keep in mind, none of these are optimized and there are some types
of regular expressions that will cause exponential time to compute, but these
are easily fixed with some simple bail-out rules. I have not implemented any
of those fixes here.

#Algorithm
The algorithm is a simple two step process:

* Take the derivative with respect to each character you are matching in order
* Does the final language accept $$\epsilon$$ (which is the empty-string language)

##A derivative of a language
In formal theory, a language is a set of characters, what we'd call a set of
strings like $$\{\text{foo},\text{bar},\text{baz}\}$$. The derivative of that
language in terms of the character *b* is
$$\delta(b)=\{\text{foo},\text{bar},\text{baz}\}=\{\text{ar},\text{az}\}$$.

##Nullability
A nullable language is one that no longer accepts any input, in other words does
the language not accept $$\epsilon$$? Such a derivative of a language
looks like $$D_z\{\text{foo},\text{bar}\}$$, where *z* is not in the language
and thus is null.

##Notation
We define two things

* A function $$\delta$$ which returns $$\epsilon$$ if the argument accepts the
  empty-string and $$\emptyset$$ when it does not
* The derivative of a regular expression *re* with respect to a character as
  $$D_{char}(re)$$.

#Operations of Regular Languages

##Matches
Here we, will define our base `RegEx` object. A character matches if the
derivative of the language with respect to that character exists. This is
inherently a recursive definition, and you can tell in the `matches` that so
long as we haven't traversed the entire string to match we will continue
deriving and checking matches.

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
class RegEx
    def isNullable raise "" end
    def derive(c) raise "" end
    def matches(string)
        if string.size == 0
            self.isNullable
        else
            self.derive(string[0]).matches(string[1..-1])
        end
    end
end
{% endcodeblock %}

##Empty: Empty-Set
This, along with the empty-string language, are what we find at the depths of
our recursive match calls. If at any point in the matching process does a match
fail, the bottom of the recursion will contain `Empty`.

$$\delta(\emptyset) = \emptyset \\
D_c(\emptyset) = \emptyset$$


{% codeblock empty.py lang:python %}
class Empty(RegEx):
    def isNullable(self):
        return False
    def derive(self, c):
        return self
{% endcodeblock %}
{% codeblock empty.js lang:javascript %}
var Empty = function() {
    this.derive = function(c) {
        return new Empty;
    };
    this.isNullable = function() {
        return false;
    };
    return this;
};
Empty.prototype = new RegEx;
{% endcodeblock %}
{% codeblock empty.rb lang:ruby %}
class Empty < RegEx
    def isNullable()
        false
    end
    def derive(c)
        self
    end
end
{% endcodeblock %}

##Blank: Empty String Language
If the string is accepted (meaning it matched all the way), then it accepts
$$\epsilon$$ which is represented by the empty-string language of `Blank`.

The derivative of $$\epsilon$$ is $$\epsilon$$ and is $$\emptyset$$ with respect
to *any* character.

$$\delta(\epsilon) = \epsilon\\
D_c(\epsilon) = \emptyset$$

{% codeblock blank.py lang:python %}
class Blank(RegEx):
    def isNullable(self):
        return True
    def derive(self, c):
        return Empty()
{% endcodeblock %}
{% codeblock blank.js lang:javascript %}
var Blank = function() {
    this.derive = function(c) {
      return new Empty;
    };
    this.isNullable = function() {
      return true;
    };
    return this;
};
Blank.prototype = new RegEx;
{% endcodeblock %}
{% codeblock blank.rb lang:ruby %}
class Blank < RegEx
    def isNullable
        true
    end
    def derive(c)
        Empty.new
    end
end
{% endcodeblock %}

##Primitive: Single Character Language
A primitive is a single character language, like {'c'}. You can think of a
string as being one or more `Primitive` languages, e.g. `'cat' = 'c' 'a' 't'`
all concatenated together.

So, the derivative of a primitive is $$\emptyset$$
and the derivative of the *re* with respect to *c* is $$\epsilon$$ if *c* is the
same as the parameter *c'* and $$\emptyset$$ if they are not equal.

$$\delta(c) = \emptyset\\
D_c(c) = \epsilon\text{ if }c=c'\\
D_c(c') = \emptyset\text{ if }c\ne c'
$$

{% codeblock primitive.py lang:python %}
class Primitive(RegEx):
    def __init__(self, c):
        self.c = c
    def isNullable(self):
        return False
    def derive(self, c):
        if self.c == c:
            return Blank()
        else:
            return Empty()
{% endcodeblock %}
{% codeblock primitive.js lang:javascript %}
var Primitive = function(c) {
    this.c = c;
    this.derive = function(c) {
        if (c === this.c) {
            return new Blank;
        } else {
            return new Empty;
        }
    };
    this.isNullable = function() {
        return false;
    };
};
Primitive.prototype = new RegEx;
{% endcodeblock %}
{% codeblock primitive.rb lang:ruby %}
class Primitive < RegEx
    def initialize(c)
        @c = c
    end
    def isNullable
        false
    end
    def derive(c)
        if @c == c
            Blank.new
        else
            Empty.new
        end
    end
end
{% endcodeblock %}

##Choice: ORing Languages
Choice is where you have two languages and matching either one is fine, e.g.
'foo' or 'bar' would match 'foo' and 'bar'. This is also known as the Union
operation in set theory.

The derivitive of the union of two languages is the union of their respective
derivatives. The same goes for the derivative of the *re* of those languages.

$$\delta(L_1\cup L_2) = \delta(L_1)\cup\delta(L_2)\\
D_c(L_1\cup L_2) = D_c(L_1)\cup D_c(L_2)$$
{% codeblock choice.py lang:python %}
class Choice(RegEx):
    def __init__(self, this, that):
        self.this = this
        self.that = that
    def isNullable(self):
        return self.this.isNullable() or self.that.isNullable()
    def derive(self, c):
        return Choice(self.this.derive(c), self.that.derive(c))
{% endcodeblock %}
{% codeblock choice.js lang:javascript %}
var Choice = function(thisOne, thatOne) {
    this.thisOne = thisOne;
    this.thatOne = thatOne;
    this.derive = function(c) {
        return new Choice(this.thisOne.derive(c), this.thatOne.derive(c));
    };
    this.isNullable = function() {
        return this.thisOne.isNullable() || this.thatOne.isNullable();
    };
    return this;
};
Choice.prototype = new RegEx;
{% endcodeblock %}
{% codeblock choice.rb lang:ruby %}
class Choice < RegEx
    def initialize(this, that)
        @this = this
        @that = that
    end
    def isNullable
        @this.isNullable || @that.isNullable
    end
    def derive(c)
        Choice.new(@this.derive(c), @that.derive(c))
    end
end
{% endcodeblock %}

##Repetition: Languages*
Repetition is zero or more repetitions of the language, e.g. 'foo' would match
'foofoofoo' and '' since it is matching 3 repetitions and 0 respectively.

The derivative of a repetition is $$\epsilon$$ and the derivative with of the
*re* is the derivative of the *re* concatenated with *re\**.

$$\delta(L*) = \epsilon\\
D_c(L*) = D_c(L)L*$$

{% codeblock repetition.py lang:python %}
class Repetition(RegEx):
    def __init__(self, base):
        self.base = base
    def isNullable(self):
        return True
    def derive(self, c):
        return Sequence(self.base.derive(c), self)
{% endcodeblock %}
{% codeblock repetition.js lang:javascript %}
var Repetition = function(internal) {
    this.internal = internal;
    this.derive = function(c) {
        return new Sequence(this.internal.derive(c), this);
    };
    this.isNullable = function() {
        return true;
    };
    return this;
};
Repetition.prototype = new RegEx;
{% endcodeblock %}
{% codeblock repetition.rb lang:ruby %}
class Repetition < RegEx
    def initialize(base)
        @base = base
    end
    def isNullable
        true
    end
    def derive(c)
        Sequence.new(@base.derive(c), self)
    end
end
{% endcodeblock %}

##Sequence: Concatenation of Languages
Like I explained earlier, strings longer than 1 character can be thought of as
concatenations of `Primitive` languages, e.g. 'foo' is the same as 'f' 'o' 'o'.

The derivative of a sequence of languages is the sequence of the derivative of
those languages. And the derivative of the *re* is the `Choice` of the sequence
of first derivative with the second language, and the derivative of the second.

$$\delta(L_1 L_2) = \delta(L_1)\delta(L_2)\\
D_c(L_1 L_2) = \delta(L_1)D_c(L_2)\cap D_c(L_1)L_2$$

{% codeblock sequence.py lang:python %}
class Sequence(RegEx):
    def __init__(self, left, right):
        self.left = left
        self.right = right
    def isNullable(self):
        return self.left.isNullable() and self.right.isNullable()
    def derive(self, c):
        if (self.left.isNullable()):
            return Choice(Sequence(self.left.derive(c), self.right),
                          self.right.derive(c))
        else:
            return Sequence(self.left.derive(c), self.right)
{% endcodeblock %}
{% codeblock sequence.js lang:javascript %}
var Sequence = function(first, second) {
    this.first = first;
    this.second = second;
    this.derive = function(c) {
        if (this.first.isNullable()) {
            return new Choice(new Sequence(this.first.derive(c), this.second),
                                           this.second.derive(c));
        } else {
            return new Sequence(this.first.derive(c), this.second);
        }
    };
    this.isNullable = function() {
        return this.first.isNullable() && this.second.isNullable();
    };
    return this;
};
Sequence.prototype = new RegEx;
{% endcodeblock %}
{% codeblock sequence.rb lang:ruby %}
class Sequence < RegEx
    def initialize(left, right)
        @left = left
        @right = right
    end
    def isNullable
        @left.isNullable && @right.isNullable
    end
    def derive(c)
        if @left.isNullable
            Choice.new(Sequence.new(@left.derive(c), @right), @right.derive(c))
        else
            Sequence.new(@left.derive(c), @right)
        end
    end
end
{% endcodeblock %}

##Intersection: Languages with equal strings
This is one of those operations you don't get with classic regular expression
engines in perl, python, ruby, etc. This is because the way they are written
make the *nfa* to *dfa* conversion blow up exponentially in most cases. With
derivatives, we get them cheaply.

The derivative of the intersection of languages is the intersection of their
derivatives. This is the same with respect to the *re*.

$$\delta(L_1\cap L_2) = \delta(L_!)\cap\delat(L_2)\\
D_c(L_1\cap L_2) = D_c(L1)\cap D_c(L_2)$$

{% codeblock intersection.py lang:python %}
class Intersection(RegEx):
    def __init__(self, this, that):
        self.this = this
        self.that = that
    def isNullable(self):
        return self.this.isNullable() and self.that.isNullable()
    def derive(self, c):
        return Intersection(self.this.derive(c), self.that.derive(c))
{% endcodeblock %}
{% codeblock intersection.js lang:javascript %}
var Intersection = function(first, second) {
    this.first = first;
    this.second = second;
    this.derive = function(c) {
        return new Intersection(this.first.derive(c), this.second.derive(c));
    };
    this.isNullable = function() {
        return this.first.isNullable() && this.second.isNullable();
    };
    return this;
};
Intersection.prototype = new RegEx;
{% endcodeblock %}
{% codeblock intersection.rb lang:ruby %}
class Intersection < RegEx
    def initialize(this, that)
        @this = this
        @that = that
    end
    def isNullable
        @this.isNullable && @that.isNullable
    end
    def derive(c)
        Intersection.new(@this.derive(c), @that.derive(c))
    end
end
{% endcodeblock %}

##Difference: Language A - Language B
This is another operation prohibitively expensive in classic implementations
that we get cheaply with derivatives. The difference of two languages would be
where all the strings accepted by A minus the strings accepted by language B.

The derivative of the difference of two languages is the difference of their
derivatives.

$$\delta(L_1 - L_2) = \delta(L_1) - \delta(L_2)\\
D_c(L_1 - L_2) = D_c(L_1) \cap ~D_c(L_2)$$

{% codeblock difference.py lang:python %}
class Difference(RegEx):
    def __init__(self, left, right):
        self.left = left
        self.right = right
    def isNullable(self):
        return self.left.isNullable() and not self.right.isNullable()
    def derive(self, c):
        return Difference(self.left.derive(c), self.right.derive(c))
{% endcodeblock %}
{% codeblock difference.js lang:javascript %}
var Difference = function(left, right) {
    this.left = left;
    this.right = right;
    this.isNullable = function() {
        return this.left.isNullable() && !this.right.isNullable();
    };
    this.derive = function(c) {
        return new Difference(this.left.derive(c), this.right.derive(c))
    };
};
Difference.prototype = new RegEx;
{% endcodeblock %}
{% codeblock difference.rb lang:ruby %}
class Difference < RegEx
    def initialize(left, right)
        @left = left
        @right = right
    end
    def isNullable
        @left.isNullable && !@right.isNullable
    end
    def derive(c)
        Difference.new(@left.derive(c), @right.derive(c))
    end
end
{% endcodeblock %}

##Complement: Not Language you are looking for
This is another example of an operation we get cheaply with derivatives, which
we don't get at all in classic implementations. The complement of a language is
where we want to match all strings that are *not* accepted.

The derivative of the complement of a language is the compliment of the
derivative of the language. Same for the *re*.

$$\delta(~L) = ~\delta(L)\\
D_c(~L) = ~D_c(L)$$
{% codeblock complement.py lang:python %}
class Complement(RegEx):
    def __init__(self, base):
        self.base = base
    def isNullable(self):
        return not base.isNullable()
    def derive(self, c):
        return Complement(self.base.derive(c))
{% endcodeblock %}
{% codeblock complement.js lang:javascript %}
var Complement = function(lang) {
    this.lang = lang;
    this.derive = function(c) {
        return new Complement(this.lang.derive(c));
    };
    this.isNullable = function() {
        return !this.lang.isNullable();
    };
    return this;
};
Complement.prototype = new RegEx;
{% endcodeblock %}
{% codeblock complement.rb lang:ruby %}
class Complement < RegEx
    def initialize(base)
        @base = base
    end
    def isNullable
        !@base.isNullable
    end
    def derive(c)
        Complement.new(@base.derive(c))
    end
end
{% endcodeblock %}
