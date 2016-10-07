---
layout: post
title: Ruby Image Blur Code Challenge
author: Adam D
---

A good challenge for practicing 2 dimensional array manipulation.

# The Challenge

Write a class that will create a data representation of an image in the form of a 2 dimensional array of 0's and 1's forming the pixels of the image. Then create a blur method that will cause any of the 1's in the image to change the pixels to the left, right, above and below to also be 1's. This method will need to accept an argument that will specify how far to apply the blur effect.

So consider the following image.

![Image]({{ site.url }}/images/image-blur-1.png)

Find the 1's (highlighted in green) and then change the 0's around those 1's to also be 1.

![Image blur]({{ site.url }}/images/image-blur-2.png)

Passing in the argument of 2 will blur like so.

![Image blur 2]({{ site.url }}/images/image-blur-3.png)

# My Solution

My solution was to first find the location/coordinates of the 1's and save them into a new array called `blur_coords`. Then iterate over this new array and using the coordinates update the originally image array. This process will repeat based on the number passed to the method.

{% highlight ruby %}
class Image

  def initialize(array)
    @image = array
  end

  def output_image
    @image.each do |row|
      puts row.join
    end
  end

  def blur!(distance=1)
    distance.times do
      blur_coords!
    end
  end

  private

    def blur_coords!
      blur_coords = []
      @image.each_with_index do |row, i|
        row.each_with_index do |x, row_i|
          blur_coords << [i, row_i] if x == 1
        end
      end

      blur_coords.each do |coord|
        @image[coord[0]][coord[1] + 1] = 1 if coord[1] + 1 <= @image[coord[0]].length - 1
        @image[coord[0]][coord[1] - 1] = 1 if coord[1] - 1 >= 0
        @image[coord[0] + 1][coord[1]] = 1 if coord[0] + 1 <= @image.length - 1
        @image[coord[0] - 1][coord[1]] = 1 if coord[0] - 1 >= 0
      end
    end

end

image = Image.new([
  [0, 0, 0, 0, 0, 0],
  [0, 1, 0, 0, 0, 0],
  [0, 0, 0, 0, 0, 0],
  [0, 0, 0, 0, 0, 0],
  [0, 0, 0, 0, 0, 0],
  [0, 0, 0, 0, 0, 0]
])
image.blur!(2)
image.output_image
{% endhighlight %}
