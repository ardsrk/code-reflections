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

#TODO: Autoload YEAR
