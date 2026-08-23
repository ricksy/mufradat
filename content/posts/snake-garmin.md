---
title: "Snake: The Classic Arcade Game on Your Garmin Watch"
date: 2026-02-25
draft: false
tags: ["garmin", "game", "connect-iq", "snake", "arcade", "retro"]
summary: "The classic Snake game comes to Garmin watches. Guide your snake, eat food, grow longer, and chase your high score -- all on your wrist."
cover:
  image: "/images/snake/hero.jpg"
  alt: "Snake - Classic Arcade Game for Garmin Watches"
---

After [Dino Run](/posts/dino-run-garmin/) and [Floppy Bird](/posts/floppy-bird-garmin/), the next classic was obvious. Snake -- the game that turned every Nokia phone into a gaming device in the late 90s. Simple rules, instant addiction, and a perfect fit for a watch screen.

## How It Works

Your snake moves across a 15x15 grid. A pink food dot appears somewhere on the board. Eat it to score a point and grow one segment longer. The catch: you cannot stop moving, you cannot reverse, and if you hit a wall or your own tail, it is game over.

The longer you survive, the faster the snake gets. What starts as a relaxed crawl turns into a tense race against your own growing body.

![Ready screen](/images/snake/screen1.jpg)

### Controls

Snake supports two input models to work across all Garmin devices:

**Button devices:**
- **Up button**: Turn left (relative to current direction)
- **Down button**: Turn right (relative to current direction)
- **Select / Enter**: Start game or restart

**Touch devices:**
- **Tap any direction**: Snake turns that way (relative to the snake's head, not the screen center)
- **Tap anywhere**: Start or restart on the ready/game over screen

The touch controls feel natural -- tap where you want the snake to go, and it picks the best direction from the snake's current position.

![Gameplay](/images/snake/screen2.png)

## Features

- **Classic Snake gameplay**: faithful to the original -- eat, grow, survive
- **15x15 grid**: sized and centered for round watch displays
- **Two input models**: relative turning for buttons, absolute direction for touch
- **Neon visual theme**: dark navy background, bright green snake, pink food
- **Snake gradient**: the head glows bright neon green, fading to darker green toward the tail
- **Directional eyes**: two small eyes on the head that follow the current direction
- **Pulsing food**: the food dot pulses gently so you never lose track of it
- **Progressive speed**: starts at 5 moves per second, speeds up with every food eaten, maxing out around 12 moves per second
- **High score persistence**: your best score is saved between sessions
- **"New Best!" celebration**: a gold highlight on the game over screen when you beat your record
- **120+ supported devices**: from Fenix 3 all the way to Fenix 8 and beyond

## Under the Hood

Like the other games in the series, everything is drawn programmatically -- no bitmap sprites or external assets. The snake, food, grid border, and all UI elements are built from rectangles, scaling perfectly to any screen resolution from 218x218 to 454x454.

The snake is stored as an array of grid coordinates. Each tick, a new head is prepended in the current direction. If the head lands on food, the tail stays (the snake grows). Otherwise, the tail is removed (the snake moves). Simple, efficient, and easy to reason about.

Food spawning builds a list of all unoccupied cells and picks one at random, so food never appears on the snake no matter how long it gets.

The speed ramp is gentle but noticeable: the timer interval drops by 5 milliseconds per food eaten, from 200ms down to a minimum of 80ms. By the time you hit 20+ points, every move counts.

## Supported Devices

Snake supports **120+ Garmin watches** including:

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

**[Search for Snake on the Garmin Connect IQ Store](https://apps.garmin.com/)**

Search for **"Snake"** in the Connect IQ Store on your phone or at apps.garmin.com.

---

If you enjoy the game, a review on the store would be much appreciated!
