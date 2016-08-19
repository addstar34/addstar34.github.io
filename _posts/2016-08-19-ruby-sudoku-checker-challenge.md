---
layout: post
title: Ruby Sudoku Checker Code Challenge
author: Adam D
---

This came up as a code challenge on Codewars and I enjoyed it so why not blog about it. It includes learnings of Arrays and Enumerable.

My solution to the problem wasn't the best but the beauty of Codewars is I can see solutions from other programmers and learn from them. So, I'm going to breakdown one of the solutions I really liked.

# The Rules of Sudoku

The board is broken up into rows (blue), columns (green) and regions (red) and each square within these rows, columns and regions must have the numbers 1 through 9 with no duplicates.

![Sudoku board]({{ site.url }}/images/sudoku-board.png)

# The Challenge

The challenge is to write a method that will take the board as a parameter and check if the sudoku puzzle is correct or not.

In short you need to check every row, column and region has the numbers 1 through 9, with no duplicates.

The sudoku board will be represented as an array of arrays like so:

{% highlight ruby %}
[
[5, 3, 4, 6, 7, 8, 9, 1, 2],
[6, 7, 2, 1, 9, 5, 3, 4, 8],
[1, 9, 8, 3, 4, 2, 5, 6, 7],
[8, 5, 9, 7, 6, 1, 4, 2, 3],
[4, 2, 6, 8, 5, 3, 7, 9, 1],
[7, 1, 3, 9, 2, 4, 8, 5, 6],
[9, 6, 1, 5, 3, 7, 2, 8, 4],
[2, 8, 7, 4, 1, 9, 6, 3, 5],
[3, 4, 5, 2, 8, 6, 1, 7, 9]
]
{% endhighlight %}

# The Solution

{% highlight ruby %}
class Array
  def sum
    self.inject { |a, i| a + i }
  end
end

def sudoku_checker(board)
  board.each_with_index do |line, x|
    return 'Try again!' if line.sum != 45

    if [0,3,6].include?(x)
      [0,3,6].each do |y|
      	region = board[x][y, 3] + board[x + 1][y, 3] + board[x + 2][y, 3]
        return 'Try again!' if region.sum != 45
      end
    end
  end

  board.transpose.each do |line|
    return 'Try again!' if line.sum != 45
  end

  'Finished!'
end

{% endhighlight %}

# The Breakdown

The best way to look at any large problem or task is to break it down into smaller parts. So, looking at the code we can see 2 main sections:

**1. The Array class**  
**2. The `sukoku_checker` method**

Then within the `sudoku_checker` method we can break that down into 2 sections as well:

**2.1 Code block `board.each_with_index`**  
**2.2 Code block `board.transpose.each`**


## 1. The Array class

{% highlight ruby %}
class Array
  def sum
    self.inject { |a, i| a + i }
  end
end
{% endhighlight %}

This is opening up the Array class and creating a new method for it called sum. This method takes self (which will be an array as we’ve added this new method to the array class) and calls the inject method on it. The inject method comes from the enumerable module which is included in the Array class by default.

The inject method iterates over the array running the block. It has 2 `block parameters`. The first is an accumulator and the second will be the value of each array element as it iterates over them. The accumulator parameter stores the result of the block and passes it to the next iteration.

A whole post could be written on the `inject` method but in short and in this case it will sum the values of an array.

Creating this new method gives us a nice clean way to call inject on an array as it will be used a few times in this challenge.

The reason for doing this is because the easiest way to know if a row, column or region is correct is summing the values and checking if they equal 45.

## 2. The `sudoku_checker` method

{% highlight ruby %}
def sudoku_checker(board)
  board.each_with_index do |line, x|
    return 'Try again!' if line.sum != 45

    if [0,3,6].include?(x)
      [0,3,6].each do |y|
      	region = board[x][y, 3] + board[x + 1][y, 3] + board[x + 2][y, 3]
        return 'Try again!' if region.sum != 45
      end
    end
  end

  board.transpose.each do |line|
    return 'Try again!' if line.sum != 45
  end

  'Finished!'
end
{% endhighlight %}

This method is broken down into 2 sections. The first section is iterating over the board using the enumerable method `each_with_index` and the second section is calling the array method `transpose` and then iterating over the transposed array using the array method `each`.

Lets break it down further and look at each section.

### 2.1 code block `board.each_with_index`

{% highlight ruby %}
board.each_with_index do |line, x|
  return 'Try again!' if line.sum != 45

  if [0,3,6].include?(x)
    [0,3,6].each do |y|
      region = board[x][y, 3] + board[x + 1][y, 3] + board[x + 2][y, 3]
      return 'Try again!' if region.sum != 45
    end
  end
end
{% endhighlight %}

This block of code will first check the rows, create a region and then check the region. Let's break it down!

The first part of this method is taking the board and calling the `each_with_index` enumerable method on it. The 2 `block parameters` are self explanatory, the first is the value of the array and the second is the index. So, the value will be each sub array i.e. a row and the index is each of the rows.

On the next line we are calling the previously declared sum method on the first value which is the first sub array. Then as stated above it will sum the array and boom! the first row has now been checked.

Now we're still in the first pass of `board.each_with_index do |line, x|` and the next part is the following conditional.

{% highlight ruby %}
if [0,3,6].include?(x)
  [0,3,6].each do |y|
    region = board[x][y, 3] + board[x + 1][y, 3] + board[x + 2][y, 3]
    return 'Try again!' if region.sum != 45
  end
end
{% endhighlight %}

In short this conditional creates the regions. It will do this by running 3 times as you can see from the first line `[0,3,6].include?(x)`. As `x` is the index this is saying execute the following code on index 0, 3 and 6 (remember our indices are our rows).

Now the next line is simply iterating over the array `[0,3,6]` using the array method `each` and setting `y` to the value of the array, so the block below this will execute 3 times with y being 0 then 3 and then 6.

{% highlight ruby %}
region = board[x][y, 3] + board[x + 1][y, 3] + board[x + 2][y, 3]
return 'Try again!' if region.sum != 45
{% endhighlight %}

This part of code is actually forming the regions as an array and then calls our `array.sum` method to the region. Lets break down how it creates the regions.

{% highlight ruby %}
region = board[x][y, 3] + board[x + 1][y, 3] + board[x + 2][y, 3]
{% endhighlight %}

To create the region we can see its summing the board with some parameters 3 times. Lets look at the first 2 sections which will make the 3rd section very self explanatory.

{% highlight ruby %}
board[x][y, 3]
{% endhighlight %}

This is taking the whole `board` array and getting index `[x]` which is also an array and then we go into this array using `[y, 3]` which means give me the value at position y plus the next 3 values and return these values as an array. So looking at the board array we just got the first 3 values of row 1.

Then next 2 parts of the sum are exactly the same except when we access the board array we are saying access the sub array at position `[x + 1]` and then `[x + 2]`. So this will give us the first 3 values of 2 and the first 3 values of row 3, adding them all together into an array.

We just created the first region! And like before we simply check this region by calling the sum method we created on it.

So we've just created and checked the first region on the first pass of `[0,3,6].each do |y|` so now it goes for the second pass. The second pass creates a region for the first 3 rows starting at column 3 as on the second pass `y` equals 3, and then the third pass is the same starting at column 6.

Now we know how the checking of rows, creating of regions and checking of regions is happening.

### 2.2 Code block `board.transpose.each`

{% highlight ruby %}
board.transpose.each do |line|
  return 'Try again!' if line.sum != 45
end
{% endhighlight %}

This block of code will create the columns and check the columns.

To create the columns the array method `transpose` is called on the `board` array. The `transpose` method assumes an arrray of arrays and transposes the rows and columns. A gotcha when using `transpose` is that the length of sub arrays must match or an error will be raised.

After the transposing we simply iterate over each column and call our `sum` method.

**Sudoku checker complete!**

I really enjoyed this code challenge and will continue to sharpen my coding skills.
