---
title: "Dino Run 0.2.0: What Fourteen Reviews Changed"
date: 2026-09-05
draft: false
tags: ["garmin", "game", "connect-iq", "dino", "testing"]
summary: "Six of fourteen reviewers asked how to duck. The answer turned out to be that on some watches, you couldn't."
cover:
  image: "/images/dino-run-update/hero.png"
  alt: "Dino Run - jump, duck, survive"
---

Dino Run went on the Connect IQ store in February and settled at 4.36 stars
across fourteen reviews. Good enough to leave alone, which is what I did for
six months.

Then I read the reviews properly, and one of them was not a compliment about
difficulty. It was a bug report I had failed to recognise.

## Six people asked the same question

I wrote a small tool to pull reviews from the store's public endpoint and group
them by theme. The result was blunt:

| Theme | Reviews |
|---|---|
| "How do I duck?" | **6 of 14** |
| Birds appear too late | 3 |
| Gets boring after a minute | 3 |
| No instructions | 2 |

Six people out of fourteen could not work out how to duck. My first instinct
was that the game needed better instructions.

That was wrong. I went looking in the code and found this: ducking was wired
only to the physical DOWN button. Tapping the screen always jumped. On a
vivoactive or a venu -- watches where the touchscreen *is* the interface -- there
was no way to duck at all.

Which meant the high-flying pterodactyls, the ones you are supposed to duck
under, were unavoidable. Not hard. Unavoidable. One reviewer wrote that the
game "looks terrible on vivoactive 6", and I had filed that under taste. It was
not taste. It was a player being killed by an obstacle the controls gave them no
way to survive.

## The fix, and the bug inside the fix

Ducking now works everywhere. Tap the lower half of the screen or swipe down;
tap the upper half or swipe up to jump. Both controls are spelled out on the
start screen, so nobody has to guess.

Because a tap cannot be held the way a button can, a touch duck runs on a short
timer -- about six tenths of a second, refreshed if you tap again.

That timer introduced a new bug, on watches that have *both* a touchscreen and a
DOWN button. Tap low to duck, then hold DOWN before the timer expires, and the
stale timer would fire mid-hold and stand the dino up. Into a bird. The same
unfair death I had just finished fixing, arriving through a different door.

The button now cancels any pending touch timer. It is a one-line fix for a bug
that would have been very hard to reproduce by hand.

## Which is the actual story here

I found that bug by writing tests. Before this release, Dino Run had none. Not
weak tests -- zero. No test directory, no assertions, no CI.

It now has sixteen, running headlessly in the simulator without a watch. More
useful than the count: I checked that they *work*. I reintroduced the duck bug
deliberately and re-ran them. Two failed, precisely the two written for it. A
test suite that has never been seen to fail is not evidence of anything.

Two of the sixteen test rendering, which I had assumed was impossible. Connect
IQ has no way to read pixels back, so you cannot assert on an image. But SDK
9.2.0 will hand an app a real drawing context backed by an offscreen buffer,
and from that you can ask real font metrics on a real device profile. So the
tests now measure every string on the start and game-over screens against the
actual screen width.

On an Instinct 2, at 176 pixels wide, the duck hint measures 159. It fits, with
seventeen pixels to spare. If anyone ever lengthens that string, a test fails
instead of a review arriving six months later.

Worth being precise about the limits: this catches text that does not fit. It
cannot see frame rate, it cannot see actual pixels, and it has no opinion about
whether the dino looks good.

## The rest of it

**Birds arrive at score 80 instead of 200.** Three reviewers said the game got
boring after about a minute, and they were describing the wait for anything
other than cacti.

**The dino has been redrawn.** A tapered tail instead of a shape that read as a
wing, a chunkier head, a bigger eye, and an arm that is actually attached to the
body. It costs sixteen draw calls per frame, two fewer than the version it
replaces, so nothing got slower. I did build a proper pixel-art sprite first and
then measured it at ninety-five draw calls for the dino alone. On an Instinct 2
that is a frame rate problem, so it stayed in the toolbox.

## Get it

Search for **Dino Run** in the Connect IQ Store, or at
[apps.garmin.com](https://apps.garmin.com/).

If you were one of the six people who could not find the duck button: it was not
you.
