---
layout: default
title: Artemis Everfree
---

## Highlighted Projects

- [vaportrail](https://github.com/inguardians/VaporTrail) - binary radio transmissions with a raspberry pi
- [amppaint](https://github.com/faithanalog/amppaint) - RF spectral painting with a raspberry pi
- [colorz](https://github.com/faithanalog/colorz) - z80 assembly puzzle game with a 4x3 pixel resolution
- [scala image compressor](https://github.com/faithanalog/ImageCompress) - barebones [discrete cosine transform](https://en.wikipedia.org/wiki/Discrete_cosine_transform) implementation

## Contact

{% include contact.md %}

## Testimonials

"She's like a fairy that leaves a trail of matrix letters behind her or some shit"

"When she said she was committing fashion crimes, I wasn't expecting high treason"

"She's a lil' creature!!"

## Blog

{% for post in site.posts %}- {{ post.date | date: "%Y-%m-%d" }} - [{{ post.title }}]({{ post.url }})
{% endfor %}
