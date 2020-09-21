---
layout: post
title:  "Autoloading Nested Constants in Ruby"
date:   2020-09-21 21:00:00 +0530
---

The [previous post](/blog/2020/09/constant-autoloading-in-ruby/) explained how top-level constants can be auto loaded in Ruby. All constants defined at the top-level are nested inside
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

# File: ./may.rb
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

A::B.x

{% endhighlight %}

Executing the singleton method results in NameError. Let's override `Module.const_missing` method for auto loading the module A::B from a/b.rb

But before overloading the const_missing method, we will define a method named [underscore][underscore_link] that will convert module names (A::B) to file paths (a/b).

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


Here is the definition of const_missing method.

{% highlight ruby linenos %}

# File: ./main.rb

ROOT = ENV['PWD']

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

On line 10 the file _a/b.rb_ is loaded by calling _require_. On line 17 the value of the constant MARCH defined inside A::B is returned by calling the
`const_get` method. Call to the singleton method **A::B.x** results in NameError for constant APRIL.

We will now modify the const_missing method to load the file _a.rb_ because the nested module B has access to constants defined in the enclosing class A.

Let's define a method named [parent][parent_link] for getting a reference to the parent of the module B.


{% highlight ruby linenos %}

# File: ./main.rb

class Module
  def parent_name
    name =~ /::[^:]+\Z/ ? $`.freeze : nil
  end

  def parent
    enclosing_name = parent_name
    if enclosing_name
      Object.const_get(enclosing_name)
    end
  end
end

{% endhighlight %}

On line 11, the *const_get* method is passed the argument "A". Since the class A is defined at the top-level the constant is returned.

We will now call the *parent.const_missing* method since the definition of constant APRIL could not be found in A::B.

{% highlight ruby linenos %}

# File: ./main.rb

class Module
  def const_missing(const_name)
    file_path = File.join(ROOT, self.underscore)

    if File.file?(file_path + ".rb")
      require file_path
    end

    if self.const_defined?(const_name)
      self.const_get(const_name)
    elsif parent
      parent.const_missing(const_name)
    else
      raise NameError.new("uninitialized constant #{const_name}")
    end
  end
end

{% endhighlight %}

On line 14 the call is `A.const_missing(APRIL)`. This leads to loading of the file _a.rb_ and the constant APRIL will be located inside the class A.
We still have to load one last constant.

Here is the full _main.rb_ file including changes to const_missing for loading MAY.


{% highlight ruby linenos %}

# File: ./main.rb

ROOT = ENV['PWD']

class Module
  def underscore
    return name unless /[A-Z-]|::/.match?(name)
    word = name.to_s.gsub("::".freeze, "/".freeze)
    word.downcase
  end

  def parent_name
    name =~ /::[^:]+\Z/ ? $`.freeze : nil
  end

  def parent
    enclosing_name = parent_name
    if enclosing_name
      Object.const_get(enclosing_name)
    end
  end

  def const_missing(const_name)
    if self != Object
      file_path = File.join(ROOT, self.underscore)
    else
      file_path = File.join(ROOT, const_name.downcase.to_s)
    end

    if File.file?(file_path + ".rb")
      require file_path
    end

    if self.const_defined?(const_name)
      self.const_get(const_name)
    elsif parent
      parent.const_missing(const_name)
    else
      Object.const_missing(const_name)
    end
  end
end

class A
  module B
    def self.x
      p [MARCH, APRIL, MAY]
    end
  end
end

A::B.x

{% endhighlight %}

Line 39 has been changed to call const_missing on Object when we have finished loading all files of enclosing constants. An `if` statement has been
added in the beginning of const_missing method to load the file having the same name as the missing constant if there are no enclosing constants.

## Reflections

* What is $`? How is it useful in the context of matching strings using regular expressions?
* What happens if you tried to load a constant that is not defined ( like JUNE )?
* Would constants still get auto loaded if you redefined const_missing in Object instead of in Module?

[underscore_link]: https://github.com/rails/rails/blob/5-2-stable/activesupport/lib/active_support/inflector/methods.rb#L92
[parent_link]: https://github.com/rails/rails/blob/5-2-stable/activesupport/lib/active_support/core_ext/module/introspection.rb#L34
