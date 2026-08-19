---
title: "humm2melody: Hum a Melody, Get the Notes to Play"
date: 2026-08-19
draft: false
tags: ["python", "textual", "audio", "dsp", "pitch-detection", "tui", "vibe-coding"]
summary: "A terminal app that listens to you hum and turns it into notes you can play on a keyboard. Written entirely by prompting an LLM -- including the pitch detection."
cover:
  image: "/images/humm2melody/demo.gif"
  alt: "humm2melody transcribing a hummed melody in the terminal"
---

I keep humming melodies I would like to play on a keyboard, and then losing them because I cannot work out which notes they are. So I built a terminal app that listens through the microphone and tells me.

Press **Start**, hum, press **Stop**. While you hum it shows the note it is hearing in real time. When you stop it lays the melody out as a piano-roll timeline, plus a plain list of notes you can read off and play.

![humm2melody transcribing a hummed melody](/images/humm2melody/demo.gif)

## Up Front: This Is a Vibe-Coded Project

Every line of it was written by an LLM (Claude) from conversational prompts. I described what I wanted, pushed back on what came out, and it wrote the code. I did not hand-write the pitch detection, and I could not have -- I do not know this field.

It works, and it is tested. But it has had no expert review, and the signal-processing choices were made by a model rather than by someone who does this for a living. I am publishing it because the result was more interesting than I expected, not because I am claiming it is correct.

More on how that actually went at the bottom.

## Hearing It Back

The problem with a transcription is that you cannot tell whether it is right by looking at it. So press **p** and it plays the melody back as tones, with a playhead sweeping across the timeline.

It plays the *snapped* notes -- the ones it would tell you to press -- not the raw frequency you hummed. That is the whole point: if it misheard you, you hear the mistake immediately, before you go anywhere near a keyboard.

## Every Run Is Recorded

Every recording is saved automatically into a timestamped folder, including the ones where nothing was detected. A failed transcription is exactly the thing you want to look at later.

![Renaming, loading and deleting saved runs](/images/humm2melody/sessions.gif)

Each run holds four files: the raw microphone audio, the tones it plays back, a JSON manifest of the detected notes, and a CSV of every analysis frame -- time, frequency, confidence, loudness, about 43 rows per second. That last one is the useful one. It is the detector's frame-by-frame opinion *before* smoothing threw anything away, so when a note comes out wrong you can actually see why.

You can rename runs, delete them, or load an old one back onto the timeline from a sidebar.

## Under the Hood

Pitch detection uses [YIN](http://audition.ens.fr/adc/pdf/2002_JASA_YIN.pdf), running over a 93ms window that slides forward every 23ms, at 22.05kHz. Humming is monophonic and sits roughly between 65Hz and 1200Hz, which makes it a friendly problem.

The bit I found genuinely clever: YIN takes the *first* dip below a threshold in its difference curve rather than the deepest one. That "first, not best" rule is what stops the detector from locking onto a harmonic and confidently reporting your note an octave too high.

Turning the frame-by-frame pitch track into notes is where the real work is. Raw output is far too jittery to read -- vibrato, glides between notes and the occasional slip all look like pitch changes. So it gets smoothed with a median filter, snapped to semitones, and grouped into runs.

Two things there were bugs before they were decisions, and both are the kind of thing I would never have guessed at:

**Smoothing must correct pitch, never voicing.** The median filter was bleeding notes into the surrounding silence, which inflated every duration and welded repeated notes into one. The fix is to re-apply the original silences after filtering.

**Silence and pitch need different window sizes.** You need a long window to measure a low fundamental, but using that same window to detect silence smears note boundaries by its own length -- a gap shorter than the window never looks fully silent. So `G4 G4` came back as a single long `G4`. Energy is now measured over a short slice at the window's centre, and the boundaries got sharp.

Both were caught by tests rather than by me noticing.

## What Vibe-Coding It Was Actually Like

The honest version.

The model wrote 125 tests, none of which need a microphone or a speaker. The strongest one renders notes to audio with the playback code, feeds that back through the detector, and checks the same notes come out. That round trip is what caught both bugs above -- the code looked completely reasonable in both cases.

It was also wrong in ways that needed pushing back on. It wrote tests that asserted the wrong thing and then "fixed" working code to match them -- one insisted 453Hz should be an A4 when it is genuinely 50 cents up and rounds to A#4. Twice it reported a defect in the app that turned out to be a quirk of the UI framework. Left alone, it would have happily built on top of any of that.

The thing that made it work was not the code generation. It was that everything was verifiable: the tests run without hardware, the demo mode is deterministic, and the screen captures above are scripted rather than hand-recorded, so they cannot quietly start showing a broken app. When a model can check its own work against something real, it gets somewhere. When it cannot, it produces confident nonsense.

## Get It

**[github.com/ricksy/humm2melody](https://github.com/ricksy/humm2melody)** -- MIT licensed.

```bash
brew install portaudio
git clone https://github.com/ricksy/humm2melody
cd humm2melody
uv sync
uv run humm2melody
```

There is a `--demo` flag that replays a synthetic hum through the real pipeline, so you can try it without a working microphone. That is how the GIFs above were made.

A couple of honest limitations: it is monophonic, so one voice at a time and no chords. It gives you note timings in seconds, not a quantised score with a time signature. And two identical notes in a row need a small gap between them, because the detector hears pitch, not attacks.
