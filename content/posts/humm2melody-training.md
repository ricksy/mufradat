---
title: "humm2melody, Part Two: Teaching the Voice Instead of the App"
date: 2026-08-23
draft: false
tags: ["python", "textual", "audio", "dsp", "pitch-detection", "singing", "tui", "vibe-coding"]
summary: "I built a terminal app to turn my humming into notes, got it working, and discovered I could not use it. So it grew a tab that trains the voice instead of only tolerating it."
cover:
  image: "/images/humm2melody-training/training.gif"
  alt: "The training tab: a pitch bar showing how far a sung note is from its target"
---

[Part one](https://mufradat.com/posts/humm2melody/) was about teaching a program to understand humming: the
glide gate that stopped legato singing coming out as a chromatic run, the
playback crackle that took a phone recording of my own speakers to diagnose,
and what it is like to build something in a field you do not know by prompting
an LLM through it.

This is what happened next, and it was not what I expected.

## First, Measure the Voice Rather Than Guess At It

Fourteen versions of listening better came in between, and the one that
matters here is calibration. Every threshold in the app started out hand-tuned against one person's voice, which is exactly the thing that should be measured per user instead.

So: open the **Calibrating** tab and do three short takes. Your lowest comfortable note, your highest, then a familiar tune you hear played and sing straight back. The app searches all 81 combinations of the two detection dials for the pair that best recovers the melody, and adopts it.

Singing a known tune back beats asking for a scale. There is no solfège to know and nothing to work out, and the tune's repeated notes and leap to the fifth happen to exercise precisely the two things the dials control.

Two decisions I would keep:

**Everything is compared as intervals.** A voice that cannot reach the reference octave sings the tune transposed, and that is a *correct* performance, not an error. It gets measured and reported -- "you sang it 1 octave down" -- rather than penalised.

**It refuses rather than guesses.** If no dial setting recovers the melody, nothing is adopted and it says so, and the derived numbers stay empty instead of being reported from a reading nobody trusts. A wrong calibration is worse than none.

The measured *range* is the part that earns its keep, because it is the one thing the dials cannot do. The dials tune segmentation, which runs after pitch detection, so they can never undo an octave error. A measured range narrows the detector's search window instead: for a B2-F#4 voice, 65-1200Hz becomes 82-554Hz, about 42% of the original, and a harmonic outside that window cannot be reported at all.

Drift and singing style are measured and deliberately *not* wired in, because the dial search already compensates for them and doing both would correct for the same thing twice.

## Train Your Voice

Here is the thing I did not see coming.

I built this to transcribe my humming. Once it worked well enough to be honest with me, it told me something I did not want to hear: I do not have the pitch control to use it. Then I handed the laptop to a couple of other people to test. Both got near-perfect transcriptions of longer and more complex songs, first try, with no dial-fiddling at all.

Which is a real result. Everything above makes the app better at understanding an imperfect voice. But if every note I hum lands on roughly the same pitch, the recording genuinely does not contain a melody, and there is no dial setting that finds one. The tool was fine. So the app grew a tab that trains the voice instead of only tolerating it.

![Training: hearing a target note, singing it, and being scored](/images/humm2melody-training/training.gif)

One target note at a time. Press `l` to hear it, `space` to sing it, `space` again to stop. Three exercises, in dependency order, because each one needs the one before it:

| Exercise | Skill | Why it is there |
| --- | --- | --- |
| **Hold one note** | Keep a single pitch steady | Nothing else is measurable without it |
| **Match the note** | Sing back a pitch you just heard | The skill most people are missing |
| **Climb the ladder** | Move a known distance between pitches | This is what a melody is |

`l` leaves a **held reference tone** running rather than playing the note once, so you can sing *against* it instead of from memory. That is a far easier exercise and the one worth doing first. (Headphones, though. Through speakers the microphone hears the reference as well as you, and a pitch detector cannot tell them apart -- it will happily score the tone as though you had sung it.)

A tall bar shows where your voice is against the target while you sing, so you can correct yourself mid-note instead of reading a verdict afterwards. Which sounds simple and was not. A raw pitch reading arrives about 43 times a second and any one of them can be an octave flip, a breath or a consonant. Left raw, the tip jumps a screen's height on a perfectly steady note, and all that teaches you is to ignore the bar. Two things settle it: a five-frame median before anything is scored, which cut the worst displayed jump from 1245 cents to 31, and easing on top. Only the median touches the score. The easing is cosmetic on purpose, so a smoother bar can never flatter your singing.

When you are a long way off, the scale zooms out -- 150 cents either side, then 300, 700, 1500 -- instead of pinning the tip to an edge. A tip stuck against the top says "too high" and nothing else: no gradient to follow, no way to tell whether you are getting closer, which is the entire mechanism the tab runs on. The readout also names the note you are *actually* singing, so an octave error reads as `F2` rather than making you decode `-1200¢`.

Scoring calls it on the note within **35 cents**, which is tighter than the 50 cents at which the app rounds to that note. Practising on a rounding boundary teaches you nothing. And the score weights *holding* the pitch for a second over merely touching it, roughly two to one, because a voice that crosses the right pitch on its way past has not sung the note. Retrying keeps your best score, so another go can only help.

### The Part That Got Deleted

The first version of this tab did something clever: it narrowed the pitch detector's search to a window around the target note. Reasonable-sounding, and it rules out the octave errors that are a pitch detector's worst mistakes.

It also makes the tab useless, and it took actually singing at it to see why. A voice *outside* the window cannot be reported at all -- only mis-reported at whatever the nearest edge happens to be. So it reads as "always too low", or "always too high", pinned there, with no relationship to what you are doing and nothing to correct. Being a long way off is the exact condition this tab exists to treat, so the search has to be able to reach that far.

It got removed. The narrowing survives only in calibration, where it is derived from a measured range rather than assumed from a target.

## The Rest

The rest of those fourteen versions, briefly: per-user profiles that remember your dials, voice, notation and tab. Note editing by keyboard or by clicking a note in any of the three views, with insert, delete and 50-deep undo. A piano keyboard you can play -- click keys and it builds a tune, which means you can compose without humming at all. Recordings stored as FLAC (the analysis master, must stay lossless) and playback as MP3 (regenerable from the note list, so it loses nothing), which took a 2.5s run from 354KB to 60KB. Four note-naming schemes, including German, where H is what English calls B and B means B-flat. A tempo dial that regenerates pitch per note rather than resampling, so slowing a melody down to learn it does not transpose it. And a mix dial for playing the tones over your original hum -- the most direct check there is, since tones that sit inside the hum mean it heard you right and tones that beat against it mean it did not.

![Renaming, loading and deleting saved runs](/images/humm2melody-training/sessions.gif)

Every run is still saved automatically, including the ones where nothing was detected, because a failed transcription is exactly the thing you want to look at later. Each one keeps the hum, the rendered tones, a JSON manifest, and a CSV of every analysis frame -- time, frequency, confidence, loudness, about 43 rows a second. That last file is the detector's frame-by-frame opinion *before* smoothing threw anything away, and it is what made every diagnosis in both of these posts possible.

## What This Turned Out To Be About

Part one ended on the observation that the model is fluent, plausible, and
occasionally confidently wrong, and that only a measurement from outside the
program settles it. That was the crackle: two reasonable hypotheses, both
wrong, resolved by holding a phone up to some speakers.

The training window is the same failure, and it is mine rather than the
model's. Narrowing the pitch search to a window around the target note is a
genuinely good idea, right up until you sing at it. Everything about the
reasoning was sound. It was still useless, and the only thing that showed that
was singing at it and watching what it said.

The pattern is not really about models. When work can be checked against
something real, it gets somewhere. When it cannot, it produces confident
nonsense at speed -- and so, in fairness, do I. The difference this time is
that the thing being checked against was my own voice, and it was not flattering.

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

The honest limitations: it is monophonic, so one voice at a time and no chords. It gives you note timings in seconds, not a quantised score with a time signature. There is no MIDI export yet. And two identical notes in a row need either a small gap or a clear re-attack, because the detector hears pitch, not fingers.
