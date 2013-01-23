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

$$\delta(L) =\{ \epsilon \}$$ if $$\epsilon\in L$$

$$\delta(L) = \emptyset $$ if $$\epsilon\not\in L$$

{% codeblock emptiness lang:python %}
class Empty(RegEx):
    def isNullable(self):
        return False
    def derive(self, c):
        return self
{% endcodeblock %}
{% codeblock emptiness lang:javascript %}
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
{% codeblock emptiness lang:ruby %}
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
{% codeblock blank lang:python %}
class Blank(RegEx):
    def isNullable(self):
        return True
    def derive(self, c):
        return Empty()
{% endcodeblock %}
{% codeblock blank lang:javascript %}
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
{% codeblock blank lang:ruby %}
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
{% codeblock primitive lang:python %}
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
{% codeblock primitive lang:javascript %}
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
{% codeblock primitive lang:ruby %}
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
{% codeblock choice lang:python %}
class Choice(RegEx):
    def __init__(self, this, that):
        self.this = this
        self.that = that
    def isNullable(self):
        return self.this.isNullable() or self.that.isNullable()
    def derive(self, c):
        return Choice(self.this.derive(c), self.that.derive(c))
{% endcodeblock %}
{% codeblock choice lang:javascript %}
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
{% codeblock choice lang:ruby %}
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
{% codeblock repetition lang:python %}
class Repetition(RegEx):
    def __init__(self, base):
        self.base = base
    def isNullable(self):
        return True
    def derive(self, c):
        return Sequence(self.base.derive(c), self)
{% endcodeblock %}
{% codeblock repetition lang:javascript %}
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
{% codeblock repetition lang:ruby %}
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
{% codeblock sequence lang:python %}
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
{% codeblock sequence lang:javascript %}
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
{% codeblock sequence lang:ruby %}
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
{% codeblock intersection lang:python %}
class Intersection(RegEx):
    def __init__(self, this, that):
        self.this = this
        self.that = that
    def isNullable(self):
        return self.this.isNullable() and self.that.isNullable()
    def derive(self, c):
        return Intersection(self.this.derive(c), self.that.derive(c))
{% endcodeblock %}
{% codeblock intersection lang:javascript %}
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
{% codeblock intersection lang:ruby %}
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
{% codeblock difference lang:python %}
class Difference(RegEx):
    def __init__(self, left, right):
        self.left = left
        self.right = right
    def isNullable(self):
        return self.left.isNullable() and not self.right.isNullable()
    def derive(self, c):
        return Difference(self.left.derive(c), self.right.derive(c))
{% endcodeblock %}
{% codeblock difference lang:javascript %}
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
{% codeblock difference lang:ruby %}
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
{% codeblock complement lang:python %}
class Complement(RegEx):
    def __init__(self, base):
        self.base = base
    def isNullable(self):
        return not base.isNullable()
    def derive(self, c):
        return Complement(self.base.derive(c))
{% endcodeblock %}
{% codeblock complement lang:javascript %}
var Compliment = function(lang) {
    this.lang = lang;
    this.derive = function(c) {
        return new Compliment(this.lang.derive(c));
    };
    this.isNullable = function() {
        return !this.lang.isNullable();
    };
    return this;
};
Compliment.prototype = new RegEx;
{% endcodeblock %}
{% codeblock complement lang:ruby %}
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
