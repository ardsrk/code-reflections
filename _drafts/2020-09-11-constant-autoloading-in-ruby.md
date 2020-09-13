---
layout: post
title:  "Constant Autoloading in Ruby"
date:   2020-09-11 16:00:00 +0530
---

Constants defined at the top-level are stored inside Object namespace. In the example below we define the YEAR constant at the top-level
and verify that the constant is stored inside Object.


{% highlight ruby %}

> Object.constants.grep(/YEAR/)
=> []

> YEAR = 2020

> Object::YEAR
=> 2020

{% endhighlight %}

Ruby calls `Object.const_missing` method if it could not locate the constant's definition. Constants can be autoloaded by overriding
this method. Here is the code to autoload the YEAR constant defined in _year.rb_ file inside the current directory.


{% highlight ruby %}

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

When Ruby executes the `puts` statement, it calls `Object.const_missing` method with YEAR as the argument. Inside `Object.const_missing` the file
_year.rb_ is loaded by calling `require`. If _year.rb_ defined the `YEAR` constant then its value is returned by calling `Object.const_get`
otherwise an error of type `NameError` is raised.

## Reflections


* What happens if you removed the last line of Object.const_missing method?
* What happens if you removed the unless statement and tried to autoload constant that is not defined ( say BEAR )?

