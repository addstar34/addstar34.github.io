---
layout: post
title: Ruby Linked List Pt2 Reverse using mutation
author: Adam D
---

In the previous blog post I reversed a linked list using a stack. In this challenge I had to reverse a linked list using a mutation to modify the initial list.

This part 2 of 3 on linked lists. See my other posts here:  
[Ruby Linked List Pt1 Reverse using a Stack]({% link _posts/2016-10-10-ruby-linked-list-reverse-using-a-stack.md %})  
[Ruby Linked List Pt3 Floyd's Cycle Detection]({% link _posts/2016-10-12-ruby-linked-list-floyds-cycle-detection.md %})

# My Solution

My solution was to use recursion and that was mainly due to the `print_value` method using recursion and staring me right in the face, but I guess an iterative approach could have been used as well. To be honest I'm not entirely sure when its best to use an iterative approach vs a recursive approach (more learning required on that).

My pseudo code was to traverse the linked list and change the `next_node` to be the previous node.

{% highlight ruby %}
def reverse_list(list, previous=nil)
  if list
    next_node = list.next_node
    list.next_node = previous
    reverse_list(next_node, list)
  end
end
{% endhighlight %}

Here we can see that I'm passing in 2 arguments; the linked list and the previous node, so when making an intial call to this method we need to start at the `head` of the linked list and not pass in a 2nd argument. The second argument will only be used during the recursion.

Then within this method I have an if statement which is used to see if we have a linked list or not and this is how we know when we've reached the end of the linked list.

Within this statement we first store the `next_node` which will become our `list` for the recursion. I then set the `list.next_node` to be the `previous` node, so this will start as nil.

The next line is where the recursion happens. Here the method `reverse_list` is calling itself and passing in the 2 arguments. The first argument I just covered and the second argument is what we want to be the previous node, which is the current `list`.

Now we can test it out using the following code which includes the linked list class and the print_values method from [Ruby Linked List Pt1 Reverse using a Stack]({% link _posts/2016-10-10-ruby-linked-list-reverse-using-a-stack.md %})

{% highlight ruby %}
class LinkedListNode
  attr_accessor :value, :next_node

  def initialize(value, next_node=nil)
    @value = value
    @next_node = next_node
  end
end

def reverse_list(list, previous=nil)
  if list
    next_node = list.next_node
    list.next_node = previous
    reverse_list(next_node, list)
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

node1 = LinkedListNode.new(37)
node2 = LinkedListNode.new(99, node1)
node3 = LinkedListNode.new(12, node2)
node4 = LinkedListNode.new(54, node3)

print_values(node4)
puts "---------"
reverse_list(node4)
print_values(node1)
{% endhighlight %}

The full code can be found on my github [here](https://github.com/addstar34/code-challenges/blob/master/linked-list/linked-list-2.rb)
