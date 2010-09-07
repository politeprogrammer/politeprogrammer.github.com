---
title: More on Rule 1
author: Jim Weirich
layout: post
---
In our last post, we introduced Mr. Manners Rule #1 of Programming Etiquette:

{% include rule01.html %}

I've got another real life example of what happens when you break that rule.

RubyGems works by replacing the system provided `require` method with
its own version.  The RubyGems version was carefully crafted to obey
the original contract as close as possible.  The only time we deviated
from the original was when the required file could not be found.
Where the original `require` method would throw an error, the RubyGems
version would use attempt to load the file from the library of gems.

The code looked something like this (taken from a old version of RubyGems):

<div class="boxed">
{% highlight ruby linenos %}
alias gem_original_require require # :nodoc:
def require(path) # :nodoc:
  gem_original_require path
rescue LoadError => load_error
  if load_error.message =~ /#{Regexp.escape path}\z/ and
     spec = Gem.searcher.find(path) then
    Gem.activate(spec.name, "= #{spec.version}")
    gem_original_require path
  else
    raise load_error
  end
end
{% endhighlight %}
</div>

Notice that we carefully use the original version of `require` (line
3) and only invoke the gem logic if the load fails with a particular
kind of load error message (line 5).

The gem specific logic finds an appropriate gem (line 6), activates it
(line 7) which essentially means that the gem is added to the load
path, and then finally re-invokes the original system require (line
8).

So, what's wrong with the above?

To answer that question, you need to look at the behavior of require
in irb.

<div class="boxed">
{% highlight ruby linenos %}
$ irb
>> require 'date'
=> true
>> require 'date'
=> false
{% endhighlight %}
</div>

The firs time `require` is invoked, it returns true because it
actually loaded the file.  The second time it is invoked, `require`
retuns false because the file is already loaded and there is no need
to load it again.

It is possible for a gem to automatically require a file when it is
activated (this autorequire feature is really frowned upon today, but
back then we didn't understand why it was a bad idea).  If the gem
loads a the same file we are require, line 8 of the RubyGems code
above will return false (because the file is already loaded).  But to
the user it looks like this:

<div class="boxed">
{% highlight ruby linenos %}
$ irb
>> require 'date'
=> false
{% endhighlight %}
</div>

It leaves the _impression_ that the file wasn't loaded.

Just to be clear, _no one_ ever depended on the true/false return
values from require, no one ever checks them.  But you _see_ the false
result in irb and people thought that RubyGems was broken and failed
to load their file.  And they would file bug reports to that effect.

So, even though the infraction against Rule #1 was minor and couldn't
possibly effect any real software, the RubyGems team made an effort to
fix the software to more closely mimic the classical version of
`require`.

Motto of the story: "Respect Rule #1".
