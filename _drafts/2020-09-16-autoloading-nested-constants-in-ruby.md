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
MAY = "May"

# File: ./a.rb
class A
  APRIL = "April"
end

# File: ./a/b.rb
class A
  module B
    MARCH = "March"
  end
end

{% endhighlight %}

MARCH is defined inside module A::B, APRIL is defined inside class A, and MAY is at the top-level. Now let's define a singleton method `x` in
module A::B in the _main.rb_ file.

{% highlight ruby %}

# File: ./main.rb
class A
  module B
    def self.x
      p [MARCH, APRIL, MAY]  # uninitialized constant A::B::MARCH (NameError)
    end
  end
end

puts A::B.x

{% endhighlight %}

Executing the singleton method results in NameError. Let's override `Module.const_missing` method for autoloading the module A::B from a/b.rb
