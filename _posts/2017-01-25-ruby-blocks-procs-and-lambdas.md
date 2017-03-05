---
layout: post
title: Ruby Blocks, Procs and Lambdas
author: Adam D
---

Understanding Ruby blocks, procs and lambdas.

# Blocks

Blocks are simply pieces of code that is supplied to a method call, also referred to as anonymous functions. They can be enclosed in `{  }` or `end  do`. Blocks can have arguments which are found between two pipes `|  |`

{% highlight ruby %}
arr = [1, 2, 3]

# single line block uses { }
arr.each { |num| puts "#{num}" }

# multiline blocks use do end
arr.each do |num|
  puts "#{num}"
end
{% endhighlight %}

Blocks work by having the method `yield` the block.

{% highlight ruby %}
def yield_a_block
  yield
end

yield_a_block { puts "hello block" }
#=> hello block
{% endhighlight %}

Yield can pass parameters to your block.

{% highlight ruby %}
def yield_a_block
  yield 2
end

yield_a_block { |num| puts "hello block " * num }
#=> hello block
#=> hello block
{% endhighlight %}

A little more complex example creating my own version of the each method.

{% highlight ruby %}
my_array = [1, 2, 3]

def my_each_method(arr)
  i = 0
  while i < arr.length
    yield arr[i]
    i += 1
  end
  arr
end

my_each_method(my_array) { |num| puts "#{num}" }
#=> 1 2 3
{% endhighlight %}


Yielding without receiving a block will error out.

{% highlight ruby %}
def yield_a_block
  yield
end

yield_a_block
#=> no block given (yield) (LocalJumpError)
{% endhighlight %}

You can use block_given? to check if a block was passed.

{% highlight ruby %}
def yield_a_block
  if block_given?
    yield
  else
    puts "no block given"
  end
end

yield_a_block
yield_a_block { puts "hello block" }
#=> no block given
#=> hello block
{% endhighlight %}

You can use implicit returns in blocks meaning the last line is returned but you cannot use explicit return within a block or it will error out.

{% highlight ruby %}
def yield_a_block
  yield
end

yield_a_block { return }
#=> unexpected return (LocalJumpError)
{% endhighlight %}


# Procs and Lambdas

Procs and lambdas are very similar to each other. They are blocks saved to a variable for later execution. To define a proc we use `Proc.new` and to define a lambda we use `lambda` or `->`. To execute a proc or lamdba we use `call`.

{% highlight ruby %}
my_proc = Proc.new { puts "hello proc" }
my_lambda = lambda { puts "hello lambda" }
my_other_lambda = -> { puts "hello other lamda" }

my_proc.call
my_lambda.call
my_other_lambda.call
#=> hello proc
#=> hello lambda
#=> hello other lambda
{% endhighlight %}

They can be passed to methods as arguments and you can even pass multiple procs or lambdas to a method (something you can't do with blocks).

{% highlight ruby %}
my_proc = Proc.new { puts "hello proc" }
my_lambda = lambda { puts "hello lambda" }
my_other_lambda = -> { puts "hello other lamda" }

def my_method(proc_arg, lambda_arg, other_lambda_arg)
  proc_arg.call
  lambda_arg.call
  other_lambda_arg.call
end

my_method(my_proc, my_lambda, my_other_lambda)
#=> hello proc
#=> hello lambda
#=> hello other lambda
{% endhighlight %}

Procs and lambdas are very similar and infact both are Proc objects.

{% highlight ruby %}
my_proc = Proc.new { puts "hello proc" }
my_lambda = lambda { puts "hello lambda" }

puts my_proc.inspect
puts my_lambda.inspect
#=> #<Proc:0x0055b36327b2f0@blocks.rb:1>
#=> #<Proc:0x0055b36327b250@blocks.rb:2 (lambda)>
{% endhighlight %}

Both can take arguments. Although notice the syntax of passing an argument to a lambda when its defined using `->`.

{% highlight ruby %}
my_proc = Proc.new { |arg| puts arg }
my_lambda = lambda { |arg| puts arg }
my_other_lambda = ->(arg) { puts arg }

my_proc.call("hello proc argument")
my_lambda.call("hello lambda argument")
my_other_lambda.call("hello other lambda argument")
#=> hello proc argument
#=> hello lambda argument
#=> hello other lambda argument
{% endhighlight %}

Procs and lambdas can be used where a block is expected by using `&` along with the proc or lambda.

{% highlight ruby %}
my_array = [1, 2, 3]
my_proc = Proc.new { |arg| puts "hello proc #{arg}" }
my_lambda = ->(arg) { puts "hello lambda #{arg}" }

my_array.each(&my_proc)
my_array.each(&my_lambda)
#=> hello proc 1
#=> hello proc 2
#=> hello proc 3
#=> hello lambda 1
#=> hello lambda 2
#=> hello lambda 3
{% endhighlight %}

and blocks can be turned into procs also by using `&`.

{% highlight ruby %}
def block_to_proc(&block)
    puts block
    block.call
end

block_to_proc { puts "I can transform into a proc" }
#=> #<Proc:0x0056100b090260@blocks.rb:6>
#=> I can transform into a proc
{% endhighlight %}

## So thats the similarities, now here's the differences

Lambdas check the number of arguments while procs do not.

{% highlight ruby %}
my_proc = Proc.new { |arg| puts arg }
my_lambda = lambda { |arg| puts arg }
my_other_lambda = ->(arg) { puts arg }

my_proc.call("hello proc argument")
my_proc.call
my_lambda.call("hello lambda argument")
my_lambda.call
my_other_lambda.call("hello other lambda argument")
my_other_lambda.call

#=> "hello proc argument"
#=>
#=> hello lambda argument
#=> wrong number of arguments (given 0, expected 1) (ArgumentError)
#=> hello other lambda argument
#=> wrong number of arguments (given 0, expected 1) (ArgumentError)
{% endhighlight %}

Lambdas and procs return differently.

A return inside a proc will return from the current context but becareful because returning from the top level context will return an error.

{% highlight ruby %}
def proc_return
  my_proc = Proc.new { return }
  my_proc.call
  puts "hello proc return here?" # this line won't be returned!
end

proc_return
#=>

my_top_level_proc = Proc.new { return }
my_top_level_proc.call
#=> unexpected return (LocalJumpError)
{% endhighlight %}

Return from inside a lambda just returns from inside the lambda.

{% highlight ruby %}
def lambda_return
  my_lambda = lambda { return }
  my_lambda.call
  puts "hello lambda return here"
end

lambda_return
#=> hello lambda return here
{% endhighlight %}

# The `method` method

It's also worth noting that normal methods can be passed to other methods similar to blocks, procs and lambdas. This is done using the `method` method and passing in the method's name as a symbol.

{% highlight ruby %}
# adjust the my_each_method from the above example to take an additional argument
def my_each_method(arr, my_method)
  i = 0
  while i < arr.length
    my_method.call(arr[i]) # use call to call the method
    i += 1
  end
  arr
end

def printer(arg)
  puts "hello #{(arg)}"
end

my_array = [1, 2, 3]
my_each_method(my_array, method(:printer))
#=> hello 1
#=> hello 2
#=> hello 3
{% endhighlight %}

# Closures

Blocks, procs and lambdas are all types of closures. A Closure is a function or reference to a function which retains access to the scope where it was defined.

{% highlight ruby %}
def my_method(lambda_arg)
  my_variable = "goodbye"
  lambda_arg.call
end

my_variable = "hello"
my_lambda = lambda { puts my_variable }

my_method(my_lambda)
#=> hello
{% endhighlight %}

Closures don't take in the actual value but instead a reference to them so if they are updated after the proc or lambda is defined it will return the lastest version of it when its called.

{% highlight ruby %}
def my_method(lambda_arg)
  my_variable = "goodbye"
  lambda_arg.call
end

my_variable = "hello"
my_lambda = lambda { puts my_variable }
my_variable += " there"

my_method(my_lambda)
#=> hello there
{% endhighlight %}
