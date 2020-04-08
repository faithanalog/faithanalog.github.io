---
layout: post
title: "Declawing the Dragon - Voice Coding in 2020"
---

In this post I am going to talk about programming with speech recognition software, also known as voice coding. Voice coding as a concept is nothing new, though you may not have heard of it. Here are some talks you can check out if you want to see what this actually look like in practice:
- [PYCON US 2013 - Using Python to Code by Voice](https://www.youtube.com/watch?v=OWyMA_bT7UI)
- [The Eleventh HOPE (2016): Coding by Voice with Open Source Speech Recognition](https://www.youtube.com/watch?v=YRyYIIFKsdU)
- [DECONSTRUCT 2019: Emily Shea - Voice Driven Development](https://www.deconstructconf.com/2019/emily-shea-voice-driven-development)

Until the past few years, one thing all of the tools for this had in common is that they relied on a piece of software called Dragon, a dictation tool developed by a company called Nuance. It's quite pricey at $300, but it's the best you can get for running dictation on your local machine. Unfortunately, it only runs on Windows; previously, there was a macOS version, but that was discontinued in 2018.

This was the big barrier for nearly every voice coding software out there. Sure, there were some tricks that allowed you to run Dragon in a virtual machine or on some other piece of hardware, but Dragon was always somewhere in the equation if you wanted something that was actually usable.

This is no longer the case! Two of the talks that I linked at the start of this post use software which does not require Dragon. The 2016 talk uses a tool called [Silvius](https://github.com/dwks/silvius), however the main website for it appears to be down at the time of writing this post, so I'm unsure of the status of this project. The video from 2019 uses an accessibility tool called [Talon](https://talonvoice.com/), which can use either Dragon or its own voice engine based on [a fork of facebook's wav2letter](https://github.com/talonvoice/wav2letter).

Talon actually works beautifully! Sure, it's not as good for dictation, but it's usable (in fact, I'm using it to write this post), and it's perfectly fine for system control and programming. Critically for me, it's also cross platform. It only works on x86_64, but it will work on macOS, Windows, and Linux. You can even host the voice processing engine on one machine and have Talon use it from another machine, allowing you to make use of the software on low power hardware.

The barrier to entry for this sort of thing is much lower than it used to be, and continues to fall. The $15/month for the Talon beta with wav2letter support can be waived for those who can't afford it, and as I mentioned, the actual voice engine itself is open source, so it's feasible for somebody to build their own software from scratch on top of it. There may be even more solutions that I'm not aware of out there.

Dragon is still the best around, but it's no longer the only option. If you decided not to try voice coding in the past because of the dependency on Dragon, I highly recommend you look into it again.