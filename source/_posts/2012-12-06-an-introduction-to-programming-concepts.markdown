---
layout: post
title: "An Introduction to Programming Concepts"
date: 2012-12-06 19:22
comments: false
published: false
categories: [programming]
---

#Introduction
I remember picking up my first saxophone. I didn't have a teacher or a book, but
I guessed at how to put it all together and started blowing. It
sounded terrible, hurt my lips, made my teeth vibrate, but it was so much fun!

After my first lesson, where I learned how to properly hold it and how to make
the correct shape with my mouth, I went home and started playing songs. Well,
not songs as most people would define them, but there were melodic things going
on.

All it took for me to start playing were a couple of concepts. I already
intuitively knew what melody and rhythm were from listening to my Michael
Jackson and Billie Holiday cassette tapes. Once I knew how to play a note on my
saxophone, I was one note closer to playing all of the notes. There is no
difference between saying to yourself, "Hey, I heard this song I like. I'm going
to figure out the notes to it" and "Hey, I installed this app I like. I'm going
to figure out how to make it".

{% codeblock How to play the saxophone lang:python %}
from sax_case import saxophone, mouthpiece, neckpiece, reed

self.wet(reed)
saxophone.attach(neckpiece)
neckpiece.attach(mouthpiece)
mouthpiece.attach(reed)

while blowing:
    play

{% endcodeblock %}

#Take the saxophone out of its case
A programming language is like an instrument, there are many to choose from
and each has their own rewards and challenges.

You might say that a language like Lisp is
like a violin - versatile and challenging; while Python is like the
saxophone - expressive and cool; C is like the piano - you can do most anything
in C; while PHP would be the slide
whistle - you can make some music on it, but it will be painful performing your
slide whistle symphony for you and your audience.

Depending on your goals, the decision might be made for you. If you want
to write an iOS app, you want to use Objective-C. If you want to write an
Android app, Java is your language.

If those decisions aren't made for you, then I would recommend Python or Ruby.
They both have great communities, growing influence in all sorts of real-world
applications, and have a relatively low learning curve.

##Think on this
PHP is one of the most widely used languages for the web today. Facebook uses
PHP and they created a project called HiPHoP to try to deal with the problems
they were facing with how many people use their service.

##Exercise
Go to your browser and search for "User Group" and a language you want to learn.
Here in Utah, we have the "Python User Group" and "Ruby User Group" to name a
few. These are great places for both inexperienced and experienced programmers
to meet and talk.

#Wet the reed
So far, we have done two simple things: taken the saxophone out of the case and
now we are going to wet the reed. You can think of these as procedures or
functions. Functions are the things programmers use to separate some action that
will be performed many times.

##Code examples
{% codeblock Python Functions lang:python %}
def wet_the_reed():
    # ...
{% endcodeblock %}
{% codeblock Ruby Functions lang:ruby %}
def wet_the_reed
    # ...
end
{% endcodeblock %}
{% codeblock Javascript Functions lang:javascript %}
function wet_the_reed() {
    // ...
}
{% endcodeblock %}
{% codeblock Perl Functions lang:perl %}
sub wet_the_read {
    # ...
}
{% endcodeblock %}
{% codeblock C Functions lang:c%}
void wet_the_reed() {
    /* ... */
}
{% endcodeblock %}
{% codeblock Java Functions lang:java%}
void wet_the_reed() {
    /* ... */
}
{% endcodeblock %}
{% codeblock Objective-C Functions lang:objectivec %}
- (void) wet_the_reed {
    // ...
}
{% endcodeblock %}
{% codeblock Scheme Functions lang:scheme %}
(define (wet_the_read)
  ... )
{% endcodeblock %}

As you can see, they all have the same pattern for defining a function
  * Some declaration that you are making a function `def`, `sub`, `define`.
  * The name of the function (what it is doing) `wet_the_reed`
  * A way to start and end the function `{}`, `()`, `end`

Ultimately, they are all doing the same thing: creating a place for you to
detail what `wet_the_reed` will do.

##Think on this
Every time you click a button on your screen, hit a key on your keyboard, move a
window on your desktop a function is called to make that happen (and probably
more than just one function).

##Exercise
What happens when you push the 'Send' button in your email program? Think about
how you can break down the things that happen when you click 'Send' into
functions.

#Attach the neck to the body


#Attach the mouthpiece to the neck

#Secure the reed to the mouthpiece

#Blow

