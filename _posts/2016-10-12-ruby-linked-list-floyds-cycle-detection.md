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

Here you can see the method sets the tortoise and hare to the first `next_node`. Then there is an `until` loop which will run until `hare.nil?` meaning until we reach the end of the linked list. So, if the list is in an infinite loop we need to make sure we return out of the loop.

Now within the loop is where we make the hare take 2 steps (1 at a time) forward by setting its value to the `next_node`, for each 1 step the tortoise takes. Then we will return if the hare reaches the end of the linked list or if the hare catches the tortoise by comparing them `if hare == tortoise`.

Now we can test it out using the following code which includes the linked list class and the print_values method from [Ruby Linked List Pt1 Reverse using a Stack]({% link _posts/2016-10-10-ruby-linked-list-reverse-using-a-stack.md %})

{% highlight ruby %}
class LinkedListNode
  attr_accessor :value, :next_node

  def initialize(value, next_node=nil)
    @value = value
    @next_node = next_node
  end
end

def print_values(list_node)
  if list_node
    print "#{list_node.value} --> "
    print_values(list_node.next_node)
  else
    print "nil\n"
    return
  end
end

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

node1 = LinkedListNode.new(37)
node2 = LinkedListNode.new(99, node1)
node3 = LinkedListNode.new(12, node2)
node4 = LinkedListNode.new(45, node3)
node5 = LinkedListNode.new(21, node4)

puts infinite_loop?(node5)
node1.next_node = node5 # creates an infinite loop
puts infinite_loop?(node5)
{% endhighlight %}

The full code can be found on my github [here](https://github.com/addstar34/code-challenges/blob/master/linked-list/linked-list-3.rb)
