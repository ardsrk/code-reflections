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

But before overloading the const_missing method, we will define a method named [underscore][underscore_link] that will convert module names (A::B) to path names (a/b).

{% highlight ruby %}

# File: ./main.rb
class Module
  def underscore
    return name unless /[A-Z-]|::/.match?(name)
    word = name.to_s.gsub("::".freeze, "/".freeze)
    word.downcase
  end
end

{% endhighlight %}

[underscore_link]: https://github.com/rails/rails/blob/5-2-stable/activesupport/lib/active_support/inflector/methods.rb#L92

Here is the definition of const_missing method.

{% highlight ruby %}

# File: ./main.rb
class Module
  def const_missing(const_name)
    file_path = File.join(ROOT, self.underscore)

    if File.file?(file_path + ".rb")
      require file_path
    end

    unless self.const_defined?(const_name)
      raise NameError.new("uninitialized constant #{self.name}::#{const_name}")
    end

    self.const_get(const_name)
  end
end

{% endhighlight %}
