---
layout: post
title:  "responsive images with jekyll"
date:   2014-01-21 12:00:00
---

Posts in [Jekyll](http://jekyllrb.com/) are generated from markdown, you can't provide any styling to images using:
`![Alt text](/path/to/img.jpg "Optional title")`. You can however use liquid tags, like `include` to help you out. 

This is how I got around it:

1) add a new `image.html` in your `_includes` directory

```html
<div class="image-wrapper">
    <img src="{{ include.url }}" alt="{{ include.description }}" />
</div>
```

2) add styling in one of your css files

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
**warning: I'm no designer**

3) whenever you would use the traditional image markdown use the include tag

```
![Octocat](https://github.global.ssl.fastly.net/images/modules/logos_page/Octocat.png "octocat")
```

```ruby
{% include image.html url="https://github.global.ssl.fastly.net/images/modules/logos_page/Octocat.png" description="octocat" %}
```

Example:
![Octocat](https://github.global.ssl.fastly.net/images/modules/logos_page/Octocat.png "octocat")

{% include image.html url="https://github.global.ssl.fastly.net/images/modules/logos_page/Octocat.png" description="octocat" %}