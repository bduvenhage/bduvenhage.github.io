---
layout: post
title:  "First post with Jekyll."
date:   2019-03-31 11:28:22 +0200
categories: jekyll
---

Jekyll is pretty cool. I followed their [Quickstart](https://github.com/jekyll/minima) guide which generates a basic blog site using the minima theme. Once you push the site to your `https://github.com/<username>/<username>.github.io` repo Github Pages will build your Jekyll site and make it available online at `<username>.github.io`.

To add a new post, one simply pushes a markdown formatted post to the `_posts` directory that follows the convention `YYYY-MM-DD-name-of-post.md` and includes the necessary front matter. Take a look at the source for this post to get an idea of what the front matter looks like.

Jekyll also offers powerful support for code snippets:

{% highlight c++ %}
#include <iostream>
  
int main(void)
{
  std::cout << "Hi, Tom.\n";
  return 0;
}
// prints "Hi, Tom." to stdout.
{% endhighlight %}

Inline tables look like so:

| Priority apples | Second priority | Third priority |
|-------|--------|---------|
| ambrosia | gala | red delicious |
| pink lady | jazz | macintosh |
| honeycrisp | granny smith | fuji |

Inline figures look like so:

<!--- - ![Logo Jekyll]({{site.url}}/assets/images/jekyll-logo.png ) -->

<!--- - ![Logo Jekyll]({{"/assets/images/jekyll-logo.png" | absolute_url}}) -->

- <img src="http://memofil.github.io/assets/images/categories/jekyll-logo.png" width="40" />

- <img src="/assets/images/jekyll-logo.png" width="40" />

<!--- - ![Logo Jekyll](/jekyll-logo.png) -->

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/

You can find the source code for Minima theme at GitHub:
[minima](https://github.com/jekyll/minima)
