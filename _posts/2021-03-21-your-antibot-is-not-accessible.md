---
layout: post
title: "Your Anti-bot is Not Accessible"
---


## TL;DR: The Thesis

Input sequencing and automation tools such as [autohotkey scripts](https://www.autohotkey.com/), hardware macros, auto-clickers, and turbo buttons are important accessibility tools that allow people with disabilities to play games they'd otherwise be unable to play. These tools are often banned in multiplayer titles, particularly MMORPGs, in the name of fairness and bot prevention. I argue that these tools should be allowed, or even implemented within the game itself. With the recognition that a line has to be drawn somewhere, I suggest that a tool should be classified as a bot only if it automatically makes meaningful decisions in response to stimuli provided directly by the game, creating a feedback loop that does not involve the player. Further, I suggest that in the games that can't allow external tools fairly, first-party accessibility features can still make the game playable for more people.


## Who needs these things?

Well, me, for one. That's why I'm writing this. I have chronic injuries in both my hands and wrists that severely limit how I can use them and how much I can use them. It sucks, and its forced changes in a lot of my habits, but I can still keep doing a lot of the things I love with the help of input automation and alternative control schemes.

So automation tools help people like me, and they more generally help anyone with limited mobility. But there's another angle here: they're preventative too! Having the kinds of injuries I have _fucking sucks_, but my injuries are caused by the proverbial death by a thousand papercuts. These tools can make games and other tasks less physically taxing by reducing those cuts by orders of magnitude, reducing the chance that someone ends up in my position at all.


## Where do the problems start?

Using automation tools in single-player games is purely a technical challenge of fitting the tool to the game. With multiplayer games though, these tools are often a grey-area in the rules or flat-out banned. I'll use World of Warcraft as an example, since its a game I like to play.

Blizzard's rule of thumb for input tools is "one user action per game action". What's pretty obviously disallowed by this is auto-clickers. I can make a game controller cast a spell when I press a button, but I can't make it cast a spell and then cast it again in 1.5 seconds, and keep going until I let go. This sucks!! It's the difference between me pressing the button once per fight or 10 times, which means I have to stop playing a lot sooner than I'd like to to avoid hurting my hands. My options for making this easier are reduced to finding buttons that hurt less to push, which has quickly diminishing returns.

There's also scenarios like combining dwell-click with eye-tracking, something a tool like [talon](https://talonvoice.com/) makes possible. In that setup, the mouse moves to where you look on screen, and then initiates a click after some delay. What even counts as a discrete user or game action? It's unfortunately not something you can get a clean cut answer on. Blizzard's support has to give vague answers or risk giving official approval for something someone else believes breaks the rules.

The problem with these rules is they prevent players from using tools that make the game more playable, because any tool that may be in the spirit of the game could still get them banned. So the choice is between not playing the game, or playing with a constant fear that your account could be banned permanently because of how you play the game.


## When is something a bot?

The point of these rules is not to prevent disabled people from playing the game. It's to prevent botting, so that in areas of competition players are competing against other players, not automation scripts. In MMORPGs this is particularly important for economic reasons, as large botnets left unchecked will flood markets with items, crashing their prices, and generate large sums of in-game money from non-player money sources to sell to players for real currency, breaking the economy even more.

So clearly you can't permit _everything_. Where do you draw the line for rules?

I think the difference is decision making. In a reductionist view of video games, a player receives a stimulus from the game, and must make a decision. This decision affects the state of the game. This prompts a new stimulus, thus completing the feedback loop.

A bot then, is a tool that removes the player from this loop. The bot receives stimuli from the game and makes meaningful decisions without a player getting involved. Banning tools by this standard allows the vast majority of accessibility tools to be used without fear.


## This probably doesn't break your game

Multiplayer games are inherently a competition of decisions. A turn-based strategy game tests long-term planning and strategy. A shooter tests reflex, coordination, tactics, and on-the-fly decisions. An MMO tests your ability to look up the optimal character build online and then do what the leveling guide/raid leader/fleet commander tells you to do (joking! but only half joking). Performing a pre-determined set of actions isn't a decision, but choosing to take those actions _is_, and that's the part the player needs to do.

However, if the optimal strategy for a game is to perform a precise pre-determined set of actions, then automation just can't apply. There's still steps you can take towards accessibility though, to add first-party options that are even better.


## Accessibility is a part of game design

The rhythm game osu! takes a reasonable compromise here. For any level, there is a pre-determined sequence of inputs that earns the best possible score. However, it has all sorts of modifiers to make the game easier. Some carry a score penalty while slowing the level down or making hit targets larger. Others automate parts of the game entirely, automatically clicking when appropriate of moving the mouse around to aim at targets. These automation modifiers invalidate the score for global leaderboards, but allow people to have fun playing levels they otherwise couldn't.

I love this concept of first-party gameplay modifiers and I'd like to see it applied more often. Modifiers that affect what reaction speed is necessary, what types of inputs are needed, how many, all of these are fantastic for making a game playable by more people.

It's the easy way out for a dev to throw their hands in the air and say "sorry, I guess you just can't play this game without breaking our rules and getting banned". Put some compassionate thought into it; you can probably allow more than you realize. And if you really can't allow automation, remember that you have the power to design a solution yourself. It will be appreciated.
