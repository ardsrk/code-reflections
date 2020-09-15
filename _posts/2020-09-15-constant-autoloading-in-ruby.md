---
layout: post
title:  "Constant Autoloading in Ruby"
date:   2020-09-11 16:00:00 +0530
---

Let's start by defining a constant in a file.

{% highlight ruby %}

# File: year.rb

YEAR = 2020

{% endhighlight %}

Now, let's attempt to access that constant from a different file.


{% highlight ruby %}

# File: main.rb

puts "The year is #{YEAR}"

{% endhighlight %}

Executing the _main.rb_ file will cause **NameError** because Ruby could not find the definition of the constant.
This can be fixed by using [Kernel.autoload](https://ruby-doc.org/core-2.7.1/Kernel.html#method-i-autoload).

{% highlight ruby %}

# File: main.rb

autoload(:YEAR, "#{ENV['PWD'}/year.rb")

puts "The year is #{YEAR}"

{% endhighlight %}

We can implement **Kernel.autoload** by overloading `Object.const_missing` method. Ruby calls *const_missing* method if it could not locate the
constant's definition. The default implementation of const_missing is to raise an exception of type **NameError**. Here is the code to autoload the
YEAR constant defined in _year.rb_ file inside the current directory.

{% highlight ruby linenos %}

ROOT = ENV['PWD']

def Object.const_missing(name)
  file_path = File.join(ROOT, name.to_s.downcase)

  if File.file?(file_path + ".rb")
    require file_path
  end

  unless const_defined?(name)
    raise NameError.new("uninitialized constant #{name}")
  end

  Object.const_get(name)
end

puts "Current year is #{YEAR}"

{% endhighlight %}

When Ruby executes the _puts_ statement, it calls `Object.const_missing` method with YEAR as the argument. Inside *const_missing* at line 7, the file
_year.rb_ is loaded by calling `require`. If _year.rb_ defined the `YEAR` constant then its value is returned by calling `Object.const_get`
otherwise an error of type `NameError` is raised.

## Reflections

* What happens if you removed the line 14 from the const_missing method?
* What happens if you removed the unless statement ( lines 10 to 12 ) and tried to autoload a constant that is not defined ( say BEAR )?

