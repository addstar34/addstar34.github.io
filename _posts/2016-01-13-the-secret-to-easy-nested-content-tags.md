---
layout: post
section-type: post
title: The Secret to Easy Nested Content_tag's
category:
tags: [ 'rails', 'content-tag', 'tables', 'secrets' ]
---

Today I’m going to show you <strong>How to Create a Table with <code>content_tag</code>'s using the Secret to Easy Nested <code>content_tag</code>‘s.

### TL;DR
Use <code>()</code> and <code>+</code> instead of <code>do</code> and <code>concat</code>.
<br />
Skip the pre-details and show me <a href="#the-secret">the secret in action</a>!

I recently needed to create a dynamic table and populate the information in the table from a hash. As we know you shouldn’t put too much logic in your views so I created a helper and a helper method and began creating the table as follows:

<pre><code data-trim class="ruby">
content_tag :table do
  content_tag :thead do
    content_tag :tr do
      content_tag :th, “HeaderCol 1”
    end
  end
end
# => "&lt;table&gt;&lt;thead&gt;&lt;tr&gt;&lt;th&gt;HeaderColumn1&lt;/th&gt;&lt;/tr&gt;&lt;/thead&gt;&lt;/table&gt;"
</code></pre>

Ok cool so far so good I thought, but then I tried to add in the table body section:

<pre><code data-trim class="ERB">
content_tag :table do
  content_tag :thead do
    content_tag :tr do
      content_tag :th, “HeaderCol 1”
    end
  end
  content_tag :tbody do
  end
end
# => "&lt;table&gt;&lt;tbody&gt;&lt;/tbody&gt;&lt;/table&gt;"
</code></pre>

Say what!? where’s my <code>thead</code>, <code>tr</code> and <code>th</code> gone! I continued to play around with it before doing a search and discovering quite a few stack overflow posts with others all having troubles creating tables using <code>content_tag</code>'s. All of the suggestions were using <code>do</code> for the nested tags, as I was using but then they were using <code>concat</code> to concatenate the <code>content_tag</code>’s to make the table populate correctly.

Off I went trying to use <code>concat</code> but it still just wasn't happening for me. Unfortunately most questions on stack overflow had people posting code answers to the the specific question but nothing with a good description of how to actually make your own table using nested <code>content_tag</code>’s.

<span id="the-secret">So, please allow me to present to you</span>

### The Secret to Easy Nested <code>content_tag</code>’s!

Ditch using do and concat and use <code>()</code> and <code>+</code>.
As before start off with the <code>table</code>, <code>thead</code>, <code>tr</code> and <code>th</code>. Let me show you:

<pre><code data-trim class="ruby">
content_tag(:table,
  content_tag(:thead,
    content_tag(:tr,
      content_tag(:th, “HeaderColumn1”)
    )
  )
)
# => "&lt;table&gt;&lt;thead&gt;&lt;tr&gt;&lt;th&gt;HeaderColumn1&lt;/th&gt;&lt;/tr&gt;&lt;/thead&gt;&lt;/table&gt;"
</code></pre>

Looking good so far, so now lets add in the <code>tbody</code> section:

<pre><code data-trim class="ruby">
content_tag(:table,
  content_tag(:thead,
    content_tag(:tr,
      content_tag(:th, “HeaderColumn1”)
    )
  ) + # notice this + to add the tbody and thead together
  content_tag(:tbody)
)
# => "&lt;table&gt;&lt;thead&gt;&lt;tr&gt;&lt;th&gt;HeaderColumn1&lt;/th&gt;&lt;/tr&gt;&lt;/thead&gt;&lt;tbody&gt;&lt;/tbody&gt;&lt;/table&gt;"
</code></pre>

Woohoo! It worked! Ok lets continue to see how to use <code>()</code> and <code>+</code> by finishing off the table:

<pre><code data-trim class="ruby">
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
# => "&lt;table&gt;&lt;thead&gt;&lt;tr&gt;&lt;th&gt;HeaderColumn1&lt;/th&gt;&lt;/tr&gt;&lt;/thead&gt;&lt;tbody&gt;&lt;tr&gt;&lt;td&gt;BodyColumn1&lt;/td&gt;&lt;/tr&gt;&lt;/tbody&gt;&lt;/table&gt;"
</code></pre>

Perfect! Let’s add some extra columns in the head and in the body to see how to bring those together using the <code>+</code>.

<pre><code data-trim class="ruby">
content_tag(:table,
  content_tag(:thead,
    content_tag(:tr,
      content_tag(:th, “HeaderColumn1”) + # notice this + to add the 2 th's together
      content_tag(:th, “HeaderColumn2”)
    )
  ) +
  content_tag(:tbody,
    content_tag(:tr,
      content_tag(:td, “BodyColumn1”) + # notice this + to add the 2 th's together
      content_tag(:td, “BodyColumn2”)
    )
  )
)
# => "&lt;table&gt;&lt;thead&gt;&lt;tr&gt;&lt;th&gt;HeaderColumn1&lt;/th&gt;&lt;th&gt;HeaderColumn2&lt;/th&gt;&lt;/tr&gt;&lt;/thead&gt;&lt;tbody&gt;&lt;tr&gt;&lt;td&gt;BodyColumn1&lt;/td&gt;&lt;td&gt;BodyColumn1&lt;/td&gt;&lt;/tr&gt;&lt;/tbody&gt;&lt;/table&gt;"
</code></pre>

Looking good but how do you add some of the extra options of the <code>content_tag</code> like giving class names to your tags. Simply find the closing <code>)</code> for the tag you wish to add the class, add the class and at the end of the line above add a <code>,</code>
Like so:

<pre><code data-trim class="ruby">
content_tag(:table,
  content_tag(:thead,
    content_tag(:tr,
      content_tag(:th, “HeaderColumn1”) +
      content_tag(:th, “HeaderColumn2”)
    )
  ) +
  content_tag(:tbody,
    content_tag(:tr,
      content_tag(:td, “BodyColumn1”) +
      content_tag(:td, “BodyColumn2”)
    )
  ), # don't forget to add this , on the line above the closing ) for the tag you want the options on
class: “terrific_table”) # this is the closing ) for the table tag add extra options here
# => "&lt;table class=\"terrific_table\"&gt;&lt;thead&gt;&lt;tr&gt;&lt;th&gt;HeaderColumn1&lt;/th&gt;&lt;th&gt;HeaderColumn2&lt;/th&gt;&lt;/tr&gt;&lt;/thead&gt;&lt;tbody&gt;&lt;tr&gt;&lt;td&gt;BodyColumn1&lt;/td&gt;&lt;td&gt;BodyColumn2&lt;/td&gt;&lt;/tr&gt;&lt;/tbody&gt;&lt;/table&gt;"
</code></pre>

Getting the hang of it now? Great! Then let’s get tricky and add a few smarts to the table using a loop to create 3 columns. I will show you how to loop through an <code>array</code> as well as using the <code>times</code> method.

<pre><code data-trim class="ruby">
header_names = %w[Mon Tue Wed]
content_tag(:table,
  content_tag(:thead,
    content_tag(:tr,
      header_names.map { |header_name| content_tag(:th, header_name ) }.join.html_safe # notice the use of .join.html_safe
    )
  ) +
  content_tag(:tbody,
    content_tag(:tr,
      3.times.map { |b| content_tag(:td, “bodyColumn#{b}”) }.join.html_safe # notice the use of join.html_safe and map
    )
  )
)
# => "&lt;table&gt;&lt;thead&gt;&lt;tr&gt;&lt;th&gt;Mon&lt;/th&gt;&lt;th&gt;Tue&lt;/th&gt;&lt;th&gt;Wed&lt;/th&gt;&lt;/tr&gt;&lt;/thead&gt;&lt;tbody&gt;&lt;tr&gt;&lt;td&gt;BodyColumn1&lt;/td&gt;&lt;td&gt;BodyColumn2&lt;/td&gt;&lt;td&gt;BodyColumn3&lt;/td&gt;&lt;/tr&gt;&lt;/tbody&gt;&lt;/table&gt;"
</code></pre>

Notice the use of the .join.html_safe at the end of the loops as well as the need for <code>map</code> on the <code>times</code> method.

So there you have it I hope this helps make your life easier when using nested <code>content_tag</code>'s. This is my first time writing a blog so please leave a comment below and let me know your thoughts; good or bad all feedback is welcomed! :)