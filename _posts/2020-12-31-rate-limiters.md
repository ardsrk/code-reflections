---
layout: post
title:  "Rate Limiters"
date:   2020-12-31 15:00:00 +0530
---

Rate limiters are used by servers to limit the usage of a service by consumers. Rate limiters ensure that the server does not get overloaded by serving
too many requests from consumers.

Let us apply this concept in Ruby. Assume that calling a method is a service and the callers of the method are consumers. Here is a method that simply
prints 10 on the terminal.

{% highlight ruby %}

def print10
  puts 10
end

{% endhighlight %}

We will need to maintain information about how many times the method has been called and whether the caller has exceeded the usage limit. For this we
will put the method in a class.


{% highlight ruby %}

class Printer
  def print10
    puts 10
  end
end

{% endhighlight %}

We will then proceed to add `@rate` and `@current_rate` variables to hold the rate limit and the current usage. For simplicity the `@rate` variable
will be hard-coded to `10` meaning that the method can be called 10 times in a minute. The eleventh call to `print10` within a minute will block and
execute after the minute has elapsed.


{% highlight ruby %}

class Printer
  def initialize
    @rate = 10
    @current_rate = 0
  end

  def print10
    if @current_rate < @rate
      puts 10
      @current_rate = @current_rate + 1
    else
      sleep 60
      @current_rate = 0
    end
  end
end

p = Printer.new
while true
  p.print10
end

{% endhighlight %}

This implementation has a short-coming. Suppose the `print10` method takes one second to execute which can be simulated by calling `sleep`. 


{% highlight ruby %}

p = Printer.new
while true
  p.print10
  sleep 1
end

{% endhighlight %}

Now when the print10 method is called the eleventh time it must block for 50 seconds. For this we will need to record the first call to print10 method
within the minute. We will use the `@time` variable for this.


{% highlight ruby %}

class Printer
  def initialize
    @rate = 10
    @current_rate = 0
  end

  def print10
    if @current_rate == 0
      @time = Time.now
      @current_rate = @current_rate + 1
      puts 10
    elsif @current_rate < @rate
      @current_rate = @current_rate + 1
      puts 10
    else
      remainder = 60 - (Time.now - @time)
      if remainder.positive?
        sleep remainder
      end
      @current_rate = 0
    end
  end
end

p = Printer.new
while true
  p.print10
  sleep 1
end

{% endhighlight %}

## Reflections

The current implementation requires the source code of the method being rate limited to be modified. Is is possible to define an API for rate limiting?
Like this:

```
  
rate_limit :print10, rate: 10

```

Try implementing the API before looking at Shopify's [Mixin](https://github.com/Shopify/limiter/blob/v1.1.0/lib/limiter/mixin.rb).
