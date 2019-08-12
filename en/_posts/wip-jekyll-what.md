---
layout: post
title:  "Jekyll -- what?"
date:   2019-07-14 17:28:10 -0300
lang: en
lang-ref: jekyll-what
---

So this is my first take ever on creating a web page.

I followed [this](https://forestry.io/blog/creating-a-multilingual-blog-with-jekyll/) tutorial by the folks at Forestry.

However their footer overites the beautiful footer created by the minimal theme. That way I learned how Jekyll works. Indeed, if I wanted to keep the theme intact but add some small changes I'd have to grab the theme's file and edit it. So I headed over to their github page to grab the default footer but I could not get it to display the footer as intended.

It was only after checking out [some more resources](https://cyberloginit.com/2018/05/05/github-pages-jekyll-minima-customize-footer.html) that I get how to set it properly. I wonder why doesn't the minima `footer.html` use the `footer-col-1`, `footer-col-2`, `footer-col-3`.

It's been very useful, despite some minor issues that got me confused.


This site was very helpful as well, for helping me set up the multilingual capabilities: https://www.sylvaindurand.org/making-jekyll-multilingual/

Inserting images: https://levimcg.com/posts/more-control-over-images-in-jekyll-posts/

Tutorial about using your gravatar
https://pt.gravatar.com/site/implement/images/

Adding images

[comment]: # (![image-title-here](/assets/favicon.png){:class="img-responsive"})
