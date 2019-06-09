---
layout: default
title: "Faith Artemis Alderson"
---

## Projects

- [brainfuck in sed]()

## Contact

- [github - faithanalog](https://github.com/faithanalog)
- [twitter - @artemis\_coven](https://twitter.com/artemis_coven)
- [telegram - @artemis\_coven](https://t.me/artemis_coven)
- [ko-fi - faithalderson](https://ko-fi.com/faithalderson)

## Blog

{% for post in site.posts %}- {{ post.date | date: "%Y-%m-%d" }} - [{{ post.title }}]({{ post.url }})
{% endfor %}
