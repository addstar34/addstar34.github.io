---
layout: post
title: Rails nested content_tag
author: Adam D
---

Recently I needed to write a rails helper to generate some html code with quite a bit of nested html. I quickly found this can be quite tricky and I discovered I wasn't alone with a quick google of terms like 'nesting content_tag', 'content_tag conact', 'content_tag table'.

So wrote a rails test and helper to play around with the outputs to get a better understanding. The best way to do this is was to try coding up a table.

I first tried writing the block with `do` and found the `content_tag`'s weren't joining:

{% highlight ruby %}
content_tag :table do
  content_tag :thead do
    content_tag :tr do
      content_tag :th, “HeaderCol 1”
    end
  end
  content_tag :tbody do
  end
end
# => "<table><tbody></tbody></table>"
{% endhighlight %}

In this example notice the output is missing `<thead><tr><th>HeaderCol 1</th></tr></thead>`. What's happening is `th`, `tr` and `thead` is generated and then `tbody` is generated after them but they aren't being joined so what get's outputted is only `tbody` within `table` as tbody is the last output in the block. I found some people using `concat` to get around this but when you start nesting more it becomes tricky.

An easier way than using concat is to write the block with `()` and use `+` to join the blocks together like so:

{% highlight ruby %}
content_tag(:table,
  content_tag(:thead,
    content_tag(:tr,
      content_tag(:th, “HeaderColumn1”)
    )
  ) +
  content_tag(:tbody,
    content_tag(:tr,
      content_tag(:td, “BodyColumn1”)
    )
  )
)
# => "<table><thead><tr><th>HeaderColumn1</th></tr></thead><tbody><tr><td>BodyColumn1</td></tr></tbody></table>"
{% endhighlight %}

While this is easier to write it can become a little hard to read when you start adding options like `class`es or `id`’s to your `content_tag` as this goes after the nested content. So I found myself writing the blocks using a combination of `do` and `()`. Here's an example of a deeply nested content_tag I wrote. It also includes some conditionals.

{% highlight ruby %}
content_tag :div, class: "row rest-avail-row" do
  content_tag :div, class: "column small-12 no-pad" do
    content_tag :div, class: "rest-avail" do
      rest_week_avail.map do |date, availability|
        content_tag :div, class: "day-avail text-center" do
          content_tag(:h4, availability[:date].strftime("%^a"), class: "bold no-marg") +
          if availability[:closed]
            content_tag(:div,
              content_tag(:h5, availability[:date].strftime("%d %^b"), class: "dark-cream bold no-marg") +
              content_tag(:h5, "N/A", class: "dark-cream"),
            class: "date-avail text-center bg-dark-cream")
          else
            if availability[:amount] == 0
              content_tag(:div,
                content_tag(:h5, availability[:date].strftime("%d %^b"), class: "gray bold no-marg") +
                content_tag(:h5, "SOLD OUT", class: "gray"),
              class: "date-avail text-center bg-medium-black")
            else
              if rest.within_cutoff_time?(Date.parse(date), availability[:time_int])
                link_to(new_restaurant_booking_path(rest, date: date)) do
                  content_tag(:div,
                    content_tag(:h5, availability[:date].strftime("%d %^b"), class: "white bold no-marg") +
                    content_tag(:h6, availability[:time], class: "white no-marg") +
                    content_tag(:h6, "#{availability[:amount]} seats", class: "white no-marg"),
                  class: "date-avail text-center bg-red")
                end
              else
                content_tag(:div,
                  content_tag(:h5, availability[:date].strftime("%d %^b"), class: "dark-cream bold no-marg") +
                  content_tag(:h6, availability[:time], class: "dark-cream no-marg"),
                class: "date-avail text-center bg-dark-cream")
              end
            end
          end
        end
      end.join.html_safe
    end
  end
end
{% endhighlight %}

And this is what the end result looked like with some pretty styling.

![My helpful screenshot]({{ site.url }}/images/availability.png)

Later I put some more thought into this and think it's better to break down the nested `content_tag`s into methods or variables and then concatenate them together. Here's an example using both methods and variables to concatenate the results together.

{% highlight ruby %}
module TableHelper
  def thead
    content_tag :thead do
      content_tag :tr do
        content_tag :th, "header col 1"
      end
    end
  end

  def tbody
    content_tag :tbody do
      tr1 = content_tag :tr do
        content_tag :td, "body col 1"
      end
      tr2 = content_tag :tr do
        content_tag :td, "body col 1 row 2"
      end
      tr1 + tr2
    end
  end

  def table
    content_tag :table do
      thead + tbody
    end
  end
end
{% endhighlight %}

I think this result could end up being a lot more tidy and readable. As always more playing and more learning to be done.
