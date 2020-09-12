---
layout: post
title:  "Constant Autoloading in Ruby"
date:   2020-09-11 16:00:00 +0530
---

Constants defined at the top-level are stored inside Object namespace. In the example below the `YEAR` constant is defined at the top-level
and we verify that the constant is stored inside `Object`.


{% highlight ruby %}

> Object.constants.grep(/YEAR/)
=> []

> YEAR = 2020

> Object::YEAR
=> 2020

{% endhighlight %}

Ruby calls `Object.const_missing` method if it could not locate a constant anywhere in the directories mentioned in the `$LOAD_PATH`.
Constants can be autoloaded by overriding `Object.const_missing` method. Here is the code to autoload the `YEAR` constant defined in current directory.


{% highlight ruby %}

ROOT = ENV['PWD']

class Object
  def self.const_missing(name)
    file_path = File.join(ROOT, name.to_s.downcase)
    path_with_suffix = file_path.sub(/(\.rb)?$/, '.rb')

    if File.file?(path_with_suffix)
      require file_path
    end

    unless const_defined?(name)
      raise NameError.new("uninitialized constant #{name}")
    end

    Object.const_get(name)
  end
end

puts "Current year is #{YEAR}"


{% endhighlight %}

#TODO: Explain the const_missing method
