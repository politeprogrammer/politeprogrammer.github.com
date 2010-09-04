---
title: On Politeness
author: Jim Weirich
---

One of the first projects I worked on after joining EdgeCase was a
Rails project that was started by a third party.  EdgeCase was brought
on to the project to help bring it to completion.  I was pairing with
Joe O'Brien ([@objo](http://twitter.com/objo)) and we had decided to
pull in some of the standard EdgeCase libraries used in other projects
to take advantage of their functionality.

Unfortunately, after bringing in those libraries, the unit tests
stopped working.  We traced the problem down to a snippet of code
similar to the following (the exact problem code is lost in the mists
of time).

<div class="boxed">
{% highlight ruby linenos %}
if "2009-02-02".match(/(\d+)-(\d+)-(\d+)$/)
  year, month, day = $1, $2, $3
  do_something_with(year, month, day)
end
{% endhighlight %}
</div>

After spending an inordinate amount of time proving that the
`do_something_with` method worked exactly as it should, we were left
with the inescapable conclusion that `String#match` was broken.

It turned out that in one of the utility libraries we added to the
project, someone thought it was a good idea to "monkey-patch"
`String#match` to do something incredibly useful.  I don't even
remember what incredibly useful thing the patched version of
`String#match` was, but there you have it.

Prior to incorporating the library into our new app, all uses of the
modified `String#match` function used the match data return value of
the method, rather than the $ variable values.  The modified version
of `String#match` _only_ returned the match data and did not set the $
variables at all (Indeed, you _can't_ set the $ variables from within
regular Ruby code).

Since our new code base depended upon `match` working the way Ruby
intended, the modified version broke the code base.  Fortunately we
were alerted to the problem immediately because of the unit tests in
the program.  It was only the dianosis of the problem that was time
consuming.

Which leads us to our first rule of polite programming ...

<div class="rule">
<div class="title">Mr. Manner's Rules of Programming Etiquette Says:</div>

<p>When replacing a function, make sure you obey the contract of the
original implementation</p>
</div>

We will talk later about what it means to "obey a contract", but for
now let's just say it means "do what the users expect".

The rules of polite programming are guidelines to producing code that
works well with other code.  In future posts we will delve into issues
of meta-programming, monkey-patching, subclassing and other areas
where "polite programming" will be a big advantage.

In the meantime, thanks for listening and may all your programming be
polite.
