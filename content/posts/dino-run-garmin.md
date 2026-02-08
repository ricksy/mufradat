---
title: "Dino Run: Chrome Dino Game on Your Garmin Watch"
date: 2026-02-05
draft: false
tags: ["garmin", "game", "connect-iq", "dino", "arcade"]
summary: "The classic Chrome offline T-Rex runner game, recreated for Garmin watches. Jump over cacti, duck under birds, and chase your high score."
cover:
  image: "/images/dino-run/hero.png"
  alt: "Dino Run - Chrome Dino Game for Garmin Watches"
---

I brought the classic Chrome offline dinosaur game to Garmin watches. If you have ever lost your internet connection and found yourself jumping a tiny T-Rex over cacti in your browser, now you can do the same thing on your wrist.

## How It Works

The game is simple: your T-Rex runs forward automatically, and obstacles appear in your path. Tap or press a button to jump over cacti. Hold down to duck under pterodactyl birds. Survive as long as you can while the game gets progressively faster.

![Gameplay](/images/dino-run/screen1.jpg)

### Controls

- **Select / Enter / Tap**: Jump (also starts and restarts the game)
- **Up button**: Jump (alternative)
- **Down button (hold)**: Duck
- **Back button**: Exit

## Features

- **3 obstacle types**: small cacti, large cacti, and pterodactyl birds
- **Progressive difficulty**: the game speeds up as your score climbs
- **Birds at score 200+**: flying at different heights -- some you jump, some you duck
- **High score persistence**: your best run is saved between sessions
- **Smooth animations**: running legs, ducking pose, ~30 FPS game loop
- **Colorful visuals**: blue sky, sandy ground, green cacti, orange birds, white clouds
- **Works on all screen shapes**: round and rectangular watch faces

![Game Over](/images/dino-run/screen2.jpg)

## Under the Hood

Everything is drawn programmatically -- no bitmap sprites. The dino is built from polygons and rectangles, making it scale perfectly to any screen size. Physics (jump velocity, gravity) also scale to screen dimensions, so the gameplay feels consistent whether you are on a small Instinct or a large Venu.

The game runs at about 30 FPS using a timer-driven loop that handles physics, obstacle spawning, animation, collision detection, and rendering each tick.

## Supported Devices

Dino Run supports **65+ Garmin watches** including:

- Fenix 7/8 series
- Forerunner 165/255/265/570/955/965/970
- Epix 2 / Epix 2 Pro
- Venu 2/3/4 series
- Vivoactive 4/5/6
- Instinct 2/3/E series
- Enduro 3
- MARQ 2 series

## Get It

**[Search for Dino Run on the Garmin Connect IQ Store](https://apps.garmin.com/)**

Search for **"Dino Run"** in the Connect IQ Store on your phone or at apps.garmin.com.

---

If you enjoy the game, a review on the store would be much appreciated!
