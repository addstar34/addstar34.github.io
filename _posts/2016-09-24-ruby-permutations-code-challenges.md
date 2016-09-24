---
layout: post
title: Ruby Permutations Code Challenges
author: Adam D
---

A couple of code challenges came up on permutations. Here are my solutions and how I went about tackling the challenges.

# Permutations Challenge 1

## Description

Create all permutations of an input string and remove duplicates, if present. This means, you have to shuffle all letters from the input in all possible orders, like so:

{% highlight ruby %}
permutations('a'); # ['a']
permutations('ab'); # ['ab', 'ba']
permutations('aabb'); # ['aabb', 'abab', 'abba', 'baab', 'baba', 'bbaa']
{% endhighlight %}

## Solution

I grabbed a pen and notebook and off I go trying to solve the solution in pseudocode. After a while I think wait up Ruby possibly already has a method for this. I check the documentation and yep it does! Once I had that the solution was easy from there.

{% highlight ruby %}
def permuatations(string)
  string.split("").permutation.to_a.map!(&:join).uniq
end
{% endhighlight %}

## After Thoughts

The String class has a `chars` method which will return an `array` of characters, so I could have used this instead of `split("")`

I could have just used `map` and not `map!`.

# Permutations Challenge 2

## Description

This problem takes its name by arguably the most important event in the life of the ancient historian Josephus: according to his tale, he and his 40 soldiers were trapped in a cave by the Romans during a siege.

Refusing to surrender to the enemy, they instead opted for mass suicide, with a twist: they formed a circle and proceeded to kill one man every three, until one last man was left (and that it was supposed to kill himself to end the act).

Well, Josephus and another man were the last two and, as we now know every detail of the story, you may have correctly guessed that they didn't exactly follow through the original idea.

You are now to create a function that returns a Josephus permutation, taking as parameters the initial array/list of items to be permuted as if they were in a circle and counted out every k places until none remained.

Tips and notes: it helps to start counting from 1 up to n, instead of the usual range 0..n-1; k will always be >=1.

For example, with n=7 and k=3 josephus(7,3) should act this way.

{% highlight ruby %}
[1,2,3,4,5,6,7] - initial sequence
[1,2,4,5,6,7] => 3 is counted out and goes into the result [3]
[1,2,4,5,7] => 6 is counted out and goes into the result [3,6]
[1,4,5,7] => 2 is counted out and goes into the result [3,6,2]
[1,4,5] => 7 is counted out and goes into the result [3,6,2,7]
[1,4] => 5 is counted out and goes into the result [3,6,2,7,5]
[4] => 1 is counted out and goes into the result [3,6,2,7,5,1]
[] => 4 is counted out and goes into the result [3,6,2,7,5,1,4]
{% endhighlight %}

So our final result is:

{% highlight ruby %}
josephus([1,2,3,4,5,6,7],3)==[3,6,2,7,5,1,4]
{% endhighlight %}

## Solution

This time I take a very quick look over the Ruby documentation array methods, then proceed to break out the pen and paper and start writing some pseduocode. My pseduocode was to continiously loop through the input array until it's length was 0, counting each item until the count is 0 then adding the item to a new array and then removing them from the input array.

Here was my solution:

{% highlight ruby %}
def josephus(items,k)
  output = []
  count = k

  until items.length == 0 do
    remove_these = []
    items.each_with_index do |item, i|
      if count == 1
        output << items[i]
        remove_these << i
      end
      count = count == 1 ? k : count -= 1
    end
    remove_these.reverse!
    remove_these.each do |r|
      items.delete_at(r)
    end
  end

  output
end
{% endhighlight %}

I was initially trying to remove items from input array based on matching but this failed a hidden test of an array of `booleans` eg `[true, false, true, true, false, true]` so I then changed to removing by index. When this hidden test ran I was like 'haha you sneaky...!' but I actually liked it as it made it more thorough.

I wasn't overly happy with my solution and when I submitted it and saw other solutions I was even less happy with it lol, and here's why:

{% highlight ruby %}
def josephus(items,k)
  Array.new(items.size) { items.rotate!(k-1).shift }
end
{% endhighlight %}

I instantly go to the ruby docs and look up the `rotate` method and have a quick play with it. After the way I solved the previous permutation problem I couldn't believe I didn't spot this method, I did take a quick look at the documentation but I think I was too eager to just smash out a solution.

## After thoughts

I didn't have the best the solution but I enjoyed the process.
