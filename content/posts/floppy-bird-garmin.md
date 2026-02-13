---
title: "Floppy Bird: Flappy Bird for Your Garmin Watch"
date: 2026-02-13
draft: false
tags: ["garmin", "game", "connect-iq", "flappy", "arcade"]
summary: "A Flappy Bird clone for Garmin watches. Tap to flap, dodge the pipes, and chase your high score -- all on your wrist."
cover:
  image: "/images/floppy-bird/hero.png"
  alt: "Floppy Bird - Tap & Fly Game for Garmin Watches"
---

After the success of [Dino Run](/posts/dino-run-garmin/), I wanted to bring another classic to Garmin watches. This time it is Flappy Bird -- the one-tap game that took over the world in 2014. Tap to flap, fly through gaps in the pipes, and try not to crash.

## How It Works

Your bird falls with gravity. Each tap gives it a flap upward. Green pipes scroll in from the right with a gap somewhere in the middle. Fly through the gap to score a point. Hit a pipe or the ground and it is game over.

Simple to learn. Surprisingly hard to put down.

![Gameplay](/images/floppy-bird/screen1.jpg)

### Controls

- **Select / Enter / Tap**: Flap (also starts and restarts the game)
- **Up button**: Flap (alternative)
- **Back button**: Exit

## Features

- **Classic one-tap gameplay**: tap to flap, gravity does the rest
- **Procedural pipes**: random gap positions every time you play
- **Progressive difficulty**: pipe speed increases as your score climbs
- **High score persistence**: your best run is saved between sessions
- **Smooth animations**: flapping wings, scrolling ground, ~30 FPS game loop
- **Colorful visuals**: blue sky, green pipes with caps, sandy ground, white clouds
- **Charming sprite**: a yellow bird with a subtle floppy disk twist
- **Works on all screen shapes**: round and rectangular watch faces

![Game Over](/images/floppy-bird/screen2.jpg)

## Under the Hood

Just like Dino Run, everything is drawn programmatically -- no bitmap sprites. The bird, pipes, ground, and clouds are all built from rectangles and lines, scaling perfectly to any screen resolution. Physics constants (gravity, flap strength, pipe speed) are derived from screen dimensions so the game feels consistent across all devices.

The difficulty is tuned for casual play on a small screen: generous pipe gaps, gentle gravity, and a soft speed ramp that caps at 2x. Forgiving hitboxes with a 2-pixel margin make near-misses feel fair.

## Supported Devices

Floppy Bird supports **120+ Garmin watches** including:

- Fenix 3/5/5+/6/7/8 series
- Forerunner 230/235/245/255/265/570/630/645/735/920/935/945/955/965/970
- Epix 2 / Epix 2 Pro
- Venu / Venu 2/3/4 series
- Vivoactive 3/4/5/6
- Instinct 2/3/E series
- Enduro / Enduro 3
- MARQ / MARQ 2 series
- Descent, D2, Approach

## Get It

**[Search for Floppy Bird on the Garmin Connect IQ Store](https://apps.garmin.com/)**

Search for **"Floppy Bird"** in the Connect IQ Store on your phone or at apps.garmin.com.

---

If you enjoy the game, a review on the store would be much appreciated!
