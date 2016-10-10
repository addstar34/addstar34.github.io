---
layout: post
title: Ruby Linked List Pt1 Reverse using a Stack
author: Adam D
---

In this challenge I learned about 2 new data structures; a linked list and a stack and how to implement them by hand. The challenge was specially to create a stack class and then reverse the linked list using this stack.

# What is a Linked List

A linked list is linear data structure that consisting of 2 elements. A data element, also known as a node, and a pointer element to the next node. The start or entry point of a linked list is called the head and last node will have a pointer of null.

![Linked List]({{ site.url }}/images/linked-list.png)

There are more complex versions of linked lists such as doubly linked list, multiply linked list and circular linked list.

# How to Implement a Linked List

Implementing a linked list in Ruby can be achieved like so:

{% highlight ruby %}
class LinkedListNode
  attr_accessor :value, :next_node

  def initialize(value, next_node=nil)
    @value = value
    @next_node = next_node
  end
end
{% endhighlight %}

Then we can build up a linked list by creating some nodes and linking them like so:

{% highlight ruby %}
node1 = LinkedListNode.new(9)
node2 = LinkedListNode.new(62, node1)
node3 = LinkedListNode.new(34, node2)
{% endhighlight %}

To output the contents of the linked list we can build a method that uses recursion to traverse the linked list like so:

{% highlight ruby %}
def print_values(list_node)
  if list_node
    print "#{list_node.value} --> "
    print_values(list_node.next_node)
  else
    print "nil\n"
    return
  end
end

node1 = LinkedListNode.new(9)
node2 = LinkedListNode.new(62, node1)
node3 = LinkedListNode.new(34, node2)

print_values(node3)

# => 34 --> 62 --> 9 --> nill
{% endhighlight %}

Fyi an iterative approach could have also been used to output the contents of the linked list.


# What is a Stack

The most common way to describe a stack is like a stack of plates. As you wash the plates you put one on top of the other. Then if you need a plate you can only remove the top plate. This is known as Last-In-First-Out (LIFO).

A stack is an abstract data type and has 2 principal operations; `push` to add element to the stack and `pop` which removes the most recently added element.

![Stack LIFO]({{ site.url }}/images/stack-lifo-1.png)


# My Stack Class Implementation

Now this is where the fun began! I had to create a Stack class and implement the 2 princiapl operations of `push` and `pop`.



# Reverse the Linked List using a Stack
