---
layout: post
title: "Adding Google Analytics to GitHub pages"
date: 2019-04-12
---

I've been wondering whether anyone actually reads this page, or whether it was just a drop
in the tech blog ocean. Resultantly, I just added Google Analytics and it couldn't be easier 
to do with Jekyll. Firstly head over to Google Analytics and sign up. Then get your tracking
tag. It should look a little like this:

```html
<!-- Global site tag (gtag.js) - Google Analytics -->
<script async src="https://www.googletagmanager.com/gtag/js?id=UA-138272437-1"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());

  gtag('config', 'UA-XXXXXX-XX');
</script>
```

Then just copy this into your `_includes` folde on your blog in a new html file. Once
you've done this you can add the following liquid tab to your default layouts. Where I've
called it `analytics.html`.

```h
{% raw  %}
{% include google_analytics.html %}
{% endraw %}
```

Google should start picking up analytics within a few hours.