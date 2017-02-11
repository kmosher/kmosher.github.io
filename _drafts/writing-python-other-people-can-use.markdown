---
title:  "Writing Python Scripts Other People Can Use"
layout: post
---

I love writing little python scripts. I _really, really_ love writing small python scripts. But there's a problem: **distributing small python scripts sucks**.

Previously, I've been living a bit of a fantasy life.
All my users were on Linux boxes that I had configuration management control over.
This meant I knew the exact environment I was targeting, and I could use `dh_virtualenv` for all my packaging.

Now though I find myself targeting a bunch of OS X users, each of whom maintains their own environment. Furthermore, I don't have easy access to a great CI pipline.


{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}



Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[virtualenv-burrito]: https://github.com/brainsik/virtualenv-burrito
[autoenv]:            https://github.com/kennethreitz/autoenv
