---
layout: post
title:  "responsive images with jekyll"
date:   2014-01-21 12:00:00
---

Posts in [Jekyll](http://jekyllrb.com/) are generated from markdown, so it can be pretty difficult to provide additional classes for styling.
You can however take advantage of liquid tags, like `include` to help you out. 

This is how I did it:

####1) add a new `image.html` in your `_includes` directory

```html
{% raw %}
<div class="image-wrapper">
    <img src="{{ include.url }}" alt="{{ include.description }}" />
</div>
{% endraw %}
```

####2) add styling in one of your css files ***(I'm no designer)***

```css
.image-wrapper{
  max-width:90%;
  height:auto;
  position: relative;
  display:block;
  margin:0 auto;
}

.image-wrapper img{
  max-width:100% !important;
  height:auto;
  display:block;
}
```

####3) whenever you would use the traditional image markdown use the include tag

```
![Octocat](https://github.global.ssl.fastly.net/images/modules/logos_page/Octocat.png "octocat")
```
would be

```ruby
{% raw %} {% include image.html url="https://github.global.ssl.fastly.net/images/modules/logos_page/Octocat.png" description="octocat" %} {% endraw %}
```

####Example (resize browser):
![Octocat](https://github.global.ssl.fastly.net/images/modules/logos_page/Octocat.png "octocat")

{% include image.html url="https://github.global.ssl.fastly.net/images/modules/logos_page/Octocat.png" description="octocat" %}