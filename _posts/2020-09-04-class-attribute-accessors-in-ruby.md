---
layout: post
title:  "Class Attribute Accessors in Ruby"
date:   2020-09-05 16:00:00 +0530
---

This post is based on active-support's [extension to attribute accessors](https://github.com/rails/rails/blob/5-2-stable/activesupport/lib/active_support/core_ext/module/attribute_accessors.rb).

Ruby provides `attr_accessor` method that defines two instance methods for each of the arguments provided.

{% highlight ruby %}

class Person
  attr_accessor :name
end

{% endhighlight %}

The `Person` class will now have two instance methods `Person#name` and `Person#name=` and an instance variable `@name`.

{% highlight ruby %}

person = Person.new
person.name = "Arvind"
puts person.name # => Arvind

{% endhighlight %}


Can we extend Ruby by adding a `cattr_accessor` method and use it to define class methods. Of course we can. 

Here is an example of its usage.

{% highlight ruby %}

class Person
  cattr_accessor :default_name
end

Person.default_name = 'noname'

{% endhighlight %}


We can open the `Class` class and define an instance method named `cattr_accessor`. Since all classes in Ruby are instances of `Class`,
the `cattr_accessor` can be called from any class.

{% highlight ruby %}

class Class

  def cattr_accessor(sym)
    class_eval %{

      @@#{sym} = nil

      def self.#{sym}
        @@#{sym}
      end

      def self.#{sym}=(val)
        @@#{sym} = val
      end
    }
  end

end

{% endhighlight %}

