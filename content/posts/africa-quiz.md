---
title: "Africa Quiz: Test Your Knowledge of African Capitals with a 3D Globe"
date: 2026-02-08
draft: false
tags: ["android", "kotlin", "jetpack-compose", "globe", "quiz", "africa"]
summary: "An offline Android quiz game covering all 54 African countries. Answer capital city questions while an interactive 3D globe spins to each location."
cover:
  image: "/images/africa-quiz/02_quiz.png"
  alt: "Africa Quiz - 3D globe with quiz question"
---

I built an Android app that quizzes you on African capital cities. It pairs each question with an interactive 3D globe that animates to the correct capital after you answer. No internet required -- everything runs offline.

## Why?

I wanted a quick way to drill African capitals and thought a spinning globe would make it more memorable than flash cards. Turns out [Globe.gl](https://globe.gl/) renders beautifully inside an Android WebView, so the whole thing came together in a single weekend.

## How It Works

Each round picks 10 countries at random from all 54 African nations. For every question you get 4 choices: the correct capital, two other African capitals, and one major non-capital city (like Lagos or Casablanca) to keep you honest.

![Home screen](/images/africa-quiz/01_home.png)

Tap **Start Quiz** and you are dropped into the game. A NASA Blue Marble globe sits in the top half of the screen, centered on Africa. Below it: the country name and four answer buttons.

![Quiz screen with 3D globe](/images/africa-quiz/02_quiz.png)

Pick an answer and the buttons lock immediately. The correct answer turns green, a wrong pick turns red, and the globe smoothly rotates and zooms to the capital's coordinates with a red dot marker.

![Answer feedback](/images/africa-quiz/03_answer_feedback.png)

After two seconds it auto-advances to the next question. At the end you get a score circle, a performance message, and a full answer-by-answer review so you can see exactly which ones you missed.

![Results screen](/images/africa-quiz/04_results.png)

There is also an optional 15-second countdown timer you can toggle from the home screen if you want some pressure.

## The Globe

The 3D globe is powered by [Globe.gl](https://globe.gl/), a Three.js-based library that makes it trivial to render an interactive Earth. I bundle the minified JS, an HTML shell, and a NASA Blue Marble texture in the app's assets folder. On Android it loads via `WebViewAssetLoader` so there are no `file:///` permission headaches.

The HTML exposes two functions -- `showCapital(lat, lng)` and `clearMarker()` -- that the Kotlin side calls through `evaluateJavascript`. That is the entire bridge between native and web.

## Tech Stack

- **Kotlin + Jetpack Compose** with Material 3
- **MVVM** architecture with `StateFlow`
- **Globe.gl** in a WebView via `WebViewAssetLoader`
- **Gson** for parsing the bundled JSON
- **Python scraper** (BeautifulSoup) to pull country data from Wikipedia, with a hardcoded fallback for all 54 countries
- Min SDK 24, fully offline, no INTERNET permission

## Get the Source

The project is open source under the MIT license:

**[codeberg.org/Mufradat/africa-quiz](https://codeberg.org/Mufradat/africa-quiz)**

Clone it, build with `./gradlew assembleDebug`, and install on any device or emulator running Android 7.0+.
