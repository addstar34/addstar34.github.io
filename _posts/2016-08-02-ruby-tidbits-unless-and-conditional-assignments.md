---
layout: post
title: Ruby Tidbits - Unless and Condition Assignments
author: Adam D
---

A short post on writing more readable and shorter code using the unless statement and conditional assignments.

## Unless Statement
Using unless instead of if can make code more readable.  
The best time to use unless is instead of writing a negative if statement.

{% highlight ruby %}
# negative if statement
if !authorized puts “access denied”

# same statement but using unless reads better
unless authorized puts “access denied”

{% endhighlight %}

 When using unless I like to read the code and see how well it flows. For example I would not use unless when testing for nil because if something reads better, also using unless probably isn’t a good idea when there’s multiple conditions.

## Shorthand Conditional Assignments
A shorthand way to write a conditional assignment while checking for a nil value is to use `||=`  
In the following example the time key in the options hash is checked and if it is nil then time is set to UTC.

{% highlight ruby %}
# long way
options[:time] = ‘UTC’ if options[:time].nil?

# short way
options[:time] ||= ‘UTC’
{% endhighlight %}

If you want to write a conditional with an if else statement this can actually be written in a long, medium and a short way!
In the following example if the user is signed in then time is set to the users time zone else its set to UTC.

{% highlight ruby %}
# long way
if user_signed_in
  options[:time] = @user.time_zone
else
  options[:time] = ‘UTC’
end

# medium way
options[:time] = if user_signed_in
  @user.time_zone
else
  ‘UTC’
end

# short way
options[:time] = user_signed_in ? @user.time_zone : ‘UTC’
{% endhighlight %}

Thanks for reading :)
