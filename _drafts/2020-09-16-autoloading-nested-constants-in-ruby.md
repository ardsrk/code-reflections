---
layout: post
title:  "Autoloading Nested Constants in Ruby"
date:   2020-09-16 11:00:00 +0530
---

The [previous post](/blog/2020/09/constant-autoloading-in-ruby/) explained how top-level constants can be autloaded in Ruby. All constants defined at the top-level are nested inside
_Object_ namespace. Here is an example.

{% highlight ruby %}

Object.constants.grep(/YEAR/)
=> []

YEAR = 2020

Object::YEAR
=> 2020

{% endhighlight %}

Let's look at an example to understand nested constants.

{% highlight ruby %}

# File: ./march.rb
MARCH = "March"

# File: ./a.rb
class A
  APRIL = "April"
end

# File: ./a/b.rb
class A
  module B
    MAY = "May"
    def self.x
      p Module.nesting       # [A::B, A]
      p [MARCH, APRIL, MAY]  # uninitialized constant A::B::MARCH (NameError)
    end
  end
end

A::B.x     

{% endhighlight %}

Executing _ruby a/b.rb_ results in NameError. Let's fix this by autoloading MARCH and APRIL in the const_missing method.

