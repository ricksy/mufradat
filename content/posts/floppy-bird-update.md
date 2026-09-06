---
title: "Floppy Bird 0.1.2: Eleven People Said It Was Slow"
date: 2026-09-07
draft: false
tags: ["garmin", "game", "connect-iq", "performance", "accessibility"]
summary: "They were right, and it was not the difficulty curve. The game was throwing away a fifth of its own speed every frame."
cover:
  image: "/images/floppy-bird-update/01_ready.jpg"
  alt: "Floppy Bird on a Garmin watch, showing the controls on the start screen"
---

Thirty-six people have reviewed Floppy Bird on the Connect IQ store. Eleven of
them said the same thing:

> "To slow" · "It's very slowly bro" · "Why is it so slow" · "Literally the
> slowest app I've ever used" · "Really fun although it moves very slow"

That is a third of all reviews, spread from one star to four. Some of the
kinder ones still recommended the game — *"moves really slow but at least I
have something to play while I don't have my phone"* — which is a generous way
to describe a problem.

I had assumed this was a difficulty complaint. Make the pipes come faster,
tune the curve, argue about game feel. One five-star reviewer even defended it:
*"I'd love to ignore all the idiots out there who can't deal with common game
designs like speed."*

Everyone was arguing about the tuning. The tuning was fine. The game simply
was not running at the speed it was tuned for.

## One line

Pipes move by a whole number of pixels each frame. The speed is not a whole
number:

```monkeyc
_baseSpeed = _w * 0.015;        // 3.9 on a 260px screen
var speedInt = _speed.toNumber();   // 3
pipe[0] = pipe[0] - speedInt;
```

`toNumber()` truncates. Every frame threw away 0.9 of a pixel and moved 3
instead of 3.9. Not once — thirty times a second, forever.

| screen | intended | actual | lost |
|---|---|---|---|
| 240px | 3.6 px/frame | 3 | **17%** |
| 260px | 3.9 px/frame | 3 | **23%** |
| 454px | 6.81 px/frame | 6 | 12% |

The smaller the watch, the more it lost. Which is exactly where the harshest
reviews came from: the Forerunner 55 and the Instinct, the cheap ones with the
small screens. *"Laggiest thing I have ever played"* was not hyperbole from
someone with a slow watch. It was the most accurate review of the lot.

The fix is to carry the remainder instead of discarding it:

```monkeyc
_carry = _carry + speed;
var whole = _carry.toNumber();
_carry = _carry - whole;
return whole;
```

Three lines. Over a hundred frames the game now covers the distance it always
intended to. It costs one floating-point addition — which matters, because the
complaint was speed and I was not about to fix it with something expensive.

## The part I would have got wrong

The obvious response to eleven "too slow" reviews is to increase the speed
constant. Someone had already started doing exactly that: an uncommitted change
raising the base speed by 25%, with a note to check the feel later.

Had that shipped on its own it would have helped, and the underlying defect
would have survived — still bleeding a fifth of the speed, still worst on the
smallest screens, just from a higher starting point. The next round of reviews
would have said the same thing more quietly.

Reading the reviews told me *what* was wrong. It could not tell me *why*, and
the difference between those two decides whether a fix lasts.

## The one-bit problem

One reviewer wrote: *"Good but not for black and white watches."* That reads
like a preference. It is not.

Garmin's cheaper watches use memory-in-pixel displays, some of which resolve
everything to black or white by luminance. Measuring what the game drew:

| element | luminance | on 1-bit |
|---|---|---|
| sky | 179.6 | white |
| pipes | 187.4 | white |
| bird | 199.9 | white |

Sky, pipes and bird all became the same white. On those watches the obstacles
were **invisible**. You were flying blind through pipes you could not see, and
the review you left was polite about it.

The pipes are now dark enough to fall on the other side of the threshold, and
the bird has a dark outline so its silhouette survives against a light sky.
Separation between sky and pipe went from 8 to 77. The cost is one extra
rectangle per frame.

![Pipes against the sky](/images/floppy-bird-update/03_pipes.jpg)

## Four people asked how to play

The start screen said *"Tap to Flap!"*. On a watch with no touchscreen that is
not merely unhelpful, it is wrong. It also never mentioned the goal.

It now says **TAP or UP to flap** and **Fly through the gaps**. My first
attempt put that text directly underneath the bird sprite, which I found by
looking at a screenshot rather than by testing — a recurring theme.

## What tests can and cannot do

Floppy Bird had no tests. It has eleven now, and the useful thing is not the
count. It is that I checked they fail: putting the truncation back breaks
exactly the four written to catch it. A test suite that has never been seen to
fail is not evidence of anything.

But no test told me whether the new speed *feels* right. Two independent speed
increases stacked here — the uncommitted 25% and my 23% recovery — and
overshooting would have traded eleven "too slow" reviews for eleven "too hard"
ones. That question was settled by a person playing it.

Reviews find the symptom. Tests pin the cause so it stays fixed. Neither one
tells you whether the game is fun.

## Get it

Search for **Floppy Bird** in the Connect IQ Store, or at
[apps.garmin.com](https://apps.garmin.com/).

If you left a review saying it was slow: you were right, it was not your watch,
and it took me too long to believe you.
