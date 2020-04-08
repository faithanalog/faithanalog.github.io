---
layout: post
title: "Declawing the Dragon - Voice Coding in 2020"
---

in this post i am going to talk about programming with speech recognition software, also known as voice coding. voice coding as a concept is nothing new, though you may not have heard of it. here are some talks you can check out if you want to see what this actually look like in practice:
- [PYCON US 2013 - Using Python to Code by Voice](https://www.youtube.com/watch?v=OWyMA_bT7UI)
- [The Eleventh HOPE (2016): Coding by Voice with Open Source Speech Recognition](https://www.youtube.com/watch?v=YRyYIIFKsdU)
- [DECONSTRUCT 2019: Emily Shea - Voice Driven Development](https://www.deconstructconf.com/2019/emily-shea-voice-driven-development)

until the past few years, one thing all of the tools for this had in common is that they relied on a piece of software called dragon, a dictation tool developed by a company called nuance. it's quite good actually, and it's the best you can get for running dictation on your local machine. unfortunately, it only runs on windows. previously, there was a mac version, but that was discontinued in 2018. it also costs the hefty sum of $300.

this was the big barrier for nearly every voice coding software out there. sure, there were some tricks that allowed you to run dragon in a virtual machine or on some other piece of hardware, but dragon was always somewhere in the equation if you wanted something that was actually usable.

this is no longer the case! two of the talks that i linked at the start of this post use software which does not require dragon. the 2016 talk uses a tool called [silvius](https://github.com/dwks/silvius), however the main website for it appears to be down at the time of writing this post, so i'm unsure of the status of this project. the video from 2019 uses an accessibility tool called [talon](https://talonvoice.com/), which, in addition to dragon, can use its own voice engine based on [a fork of facebook's wav2letter](https://github.com/talonvoice/wav2letter).

talon actually works beautifully! sure, it's not as good for dictation, but it's usable (in fact, i'm using it to write this post), and it's perfectly fine for system control and programming. critically for me, it's also cross platform. it only works on x86_64, but it will work on mac, windows, and linux. you can even host the voice processing engine on one machine and have talon use it from another machine, allowing you to make use of the software on low power hardware.

the barrier to entry for this sort of thing is much lower than it used to be, and continues to fall. the $15/month for the talon beta with wav2letter support can be waived for those who can't afford it, and as i mentioned, the actual voice engine itself is open source, so it's feasible for somebody to build their own software from scratch on top of it. there may be even more solutions that i'm not aware of out there.

dragon is still the best around, but it's no longer the only option. if you decided not to try voice coding in the past because of the dependency on dragon, i highly recommend you look into it again.