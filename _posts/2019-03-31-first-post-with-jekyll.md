---
layout: post
title:  "First post with Jekyll"
date:   2019-03-31
published: true
comments: true
categories: [jekyll]
tags: [first post, quickstart]
---

Jekyll is pretty cool. I followed the [Quickstart](https://jekyllrb.com/docs/) guide which generates a basic blog site using the minima theme. Once you push the source to your `https://github.com/<username>/<username>.github.io` repo, Github Pages will build your site and make it available online at `<username>.github.io`. You can see what the site looks like locally before pushing by running `bundle exec jekyll serve --drafts` and pointing your browser at `127.0.0.1:4000`.

To add a new post, one simply pushes a markdown formatted post to the `_posts` directory that follows the convention `YYYY-MM-DD-name-of-post.md` and includes the necessary front matter. Take a look at the [source](https://raw.githubusercontent.com/bduvenhage/bduvenhage.github.io/master/_posts/2019-03-31-first-post-with-jekyll.md) for this post to get an idea of what the front matter looks like.

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

- <img src="http://memofil.github.io/assets/images/categories/jekyll-logo.png" width="80" />

- <img src="/assets/images/jekyll-logo.png" width="80" />

<!--- - ![Logo Jekyll](/jekyll-logo.png) -->

Inline math look like so:
<!--- Put the below script somewhere. -->
<script src='https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.5/latest.js?config=TeX-MML-AM_CHTML' async></script>

$$\sum_{i=1}^m y^{(i)}$$

$$x = {-b \pm \sqrt{b^2-4ac} \over 2a}.$$

The GitHub help page on [setting up your github pages site locally with jekyll](https://help.github.com/en/articles/setting-up-your-github-pages-site-locally-with-jekyll) might be useful. Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll and the [source](https://github.com/jekyll/minima) of the minima theme to see what a complete Jekyll site looks like. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk]. 

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/

