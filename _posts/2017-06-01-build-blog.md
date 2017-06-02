---
layout: post
title: How I build this blog with Jeklly and GitHub Pages (Step-by-Step)
tags: [jekyll]
comments: True
---

<div class="message">
  This is a step-by-step record of how I build this blog with Jeklly and GitHub Pages. 
</div>

All the files to build this blog could be found at [my Github repository](https://github.com/LuciaXu/LuciaXu.github.io).

### Step 1: Fork Lanyon Theme to my Github repository.

There are lots of [Jeklly Themes](http://jekyllthemes.org/) that we could choose from. I choose [Lanyon](https://github.com/poole/lanyon) because it's the simplest one. Here are the steps to get started.

- Fork the theme repository to my own github.
- Rename the repository to *username.github.io*
- Change something in the *_config.yml* file (or any file in the repository). It'll force GitHub Pages to build the website.

Now, when I go to *http://username.github.io*, the website is there.

### Step 2: Customize the website.

- Know the basic structure of Jeklly directory.
- Install Jeklly.

To customize the website, you need to know some basic  ideas about how Jeklly site directory is structured. In Lanyon it looks like this:

{% highlight js %}
.
|
|-- _config.yml
|
|-- _includes
|       |
|       |-- head.html
|       |-- sidebar.html
|
|-- _layouts
|       |
|       |-- page.html
|       |-- post.html
|       |-- ...
|
|-- _posts
|       |
|       |-- 2014-01-01-blog.md
|       |-- ...
|
|-- index.html
{% endhighlight %}

The detail of the directory structure is [here](https://jekyllrb.com/docs/structure/). Basically, *_config.yml* stores some configuration data, like the name and url of the blog. *_layouts* files are templates of the posts. *_includes* files are partials that can be reused in layouts and posts. *_posts* files are where your blog posts are stored. *index.html* is the root directory (home) of your blog.

Every time you change something in the directory, push it to the github repository, GitHub pages will build it automatically. However, a better way of doing this, is to check if everything is what you want it to be before you push it to github. Therefore, I installed Jekyll on my laptop. (Since the os of my laptop is Windows 10, I followed [this guide](http://jekyll-windows.juthilo.com/).)

After install Jekyll, navigate into the root directory of the local Jekyll site repository and put *jekyll serve* command, then the preview of the website can be found at *http://localhost:4000* . So that every time you change something, you can review it before push it to the GitHub Pages.

### Step 3: Add an archive page to my website.

Create a markdown file in the root directory. Here's the code to build a archive page:

{% highlight js %}
---
layout: page
title: Archive
---

{{ "{% for post in site.posts " }}%}
{{ "{{ post.date | date_to_string "}}}} &nbsp;&nbsp;&raquo;&nbsp;&nbsp; [ {{ "{{ post.title """}} ]({{" {{ post.url " }}}})
{{ "{% endfor " }}%}
{% endhighlight %}

A note: I'm followed [this answer](https://stackoverflow.com/questions/3426182/how-to-escape-liquid-template-tags) to escape liquid template tags in this blog.

### Step 4: Add tags to posts, and a tag page to show posts by tags.

I followed [this tutorial](http://blog.lanyonm.org/articles/2013/11/21/alphabetize-jekyll-page-tags-pure-liquid.html) to add tags to posts, and alphabetize the tag lists on the tag page.

### Step 5: Add comment function to the posts.

I added [Disqus](https://disqus.com/) to enable comments under the blog. Here are the steps:

- Create a file *comments.html* in *_includes* directory, which contains the code below:

{% highlight js %}

{{ "{% if page.comments "}}%}
<!-- Add Disqus comments. -->
<div id="disqus_thread"></div>
<script type="text/javascript">
    /* * * CONFIGURATION VARIABLES * * */
    var disqus_shortname = 'Lucia';
    
    /* * * DON'T EDIT BELOW THIS LINE * * */
    (function() {
        var dsq = document.createElement('script'); dsq.type = 'text/javascript'; dsq.async = true;
        dsq.src = '//' + disqus_shortname + '.disqus.com/embed.js';
        (document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(dsq);
    })();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript" rel="nofollow">comments powered by Disqus.</a></noscript>
{{" {% endif "}}%}

{% endhighlight %}

- Modify the file *_layouts/post.html*. Add this line at the end of the file:
{% highlight js %}
{{"{% include comments.html "}}%}
{% endhighlight %}

- Finally, I can enable comments of a post by adding *comments:True* in the YAML header of the post.

### Step 6: Only show an excerpt of posts in Home page.

Modify the *index.html* file. Replace the `{{"{{ post.content "}}}}` line with `{{"{{ post.content | markdownify | strip_html | truncatewords: 50 "}}}}`. It'll gets the first 50 words and strips any formatting.

### Step 6: Write blogs using Markdown.
Finally, I installed [MarkdownPad](http://markdownpad.com/) to write posts. I created a directory named *_drafts* to put draft posts. Normally, they won't be presented when you do *jekyll serve*. In order to see them, use *jekyll serve --draft* instead.