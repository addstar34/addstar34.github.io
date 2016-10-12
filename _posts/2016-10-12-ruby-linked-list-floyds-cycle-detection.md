---
layout: post
title: Ruby Linked List Pt3 Floyd's Cycle Detection
author: Adam D
---

This challenge was to implement Floyds Cycle Detection algorithm also know as the Tortoise and the Hare algorithm. It's used to detect a loop in a singly linked list.

This is part 3 of 3 on my posts about linked lists. See my other posts here:  
[Ruby Linked List Pt1 Reverse using a Stack]({% link _posts/2016-10-10-ruby-linked-list-reverse-using-a-stack.md %})  
[Ruby Linked List Pt2 Reverse using mutation]({% link _posts/2016-10-11-ruby-linked-list-reverse-using-mutation.md %})

# The Tortoise and the Hare

The basic concept is the tortoise and the hare start at the head of the Linked List. Both move 1 position at a time to the next_node but the Tortoise is slower and only moves once for every 2 moves the hare makes. If the hare catches the tortoise we know there is a loop otherwise the hare will simply reach the end of the linked list.

Read more about Floyds Tortoise and the Hare on [wikipedia](https://en.wikipedia.org/wiki/Cycle_detection#Tortoise_and_hare)

# My implementation

{% highlight ruby %}
def infinite_loop?(list)
  tortoise = list.next_node
  hare = list.next_node

  until hare.nil?
    hare = hare.next_node
    return false if hare.nil?

    hare = hare.next_node
    tortoise = tortoise.next_node
    return true if hare == tortoise
  end

  return false
end
{% endhighlight %}



The full code can be found on my github [here](https://github.com/addstar34/code-challenges/blob/master/linked-list/linked-list-3.rb)
