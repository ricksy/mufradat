---
title: "humm2melody: Hum a Melody, Get the Notes to Play"
date: 2026-08-23
draft: false
tags: ["python", "textual", "audio", "dsp", "pitch-detection", "singing", "tui", "vibe-coding"]
summary: "A terminal app that listens to you hum and turns it into notes you can play on a keyboard. Fifteen versions on, it also trains the voice doing the humming. Written entirely by prompting an LLM."
cover:
  image: "/images/humm2melody/demo.gif"
  alt: "humm2melody transcribing a hummed melody in the terminal"
---

I keep humming melodies I would like to play on a keyboard, and then losing them because I cannot work out which notes they are. So I built a terminal app that listens through the microphone and tells me.

Press **Start**, hum, press **Stop**. While you hum it shows the note it is hearing in real time. When you stop it lays the melody out four ways at once: a piano-roll timeline, a plain list of notes you can read off, a piano keyboard showing where your hands go, and a table with each note's timing and tuning.

![humm2melody transcribing a hummed melody](/images/humm2melody/demo.gif)

I wrote about v0.1 here a while ago. It is at v0.15.0 now, and the interesting part is not the pile of features. It is that building a tool to understand my humming taught me that my humming was the problem.

## Up Front: This Is a Vibe-Coded Project

Every line of it was written by an LLM (Claude) from conversational prompts. I described what I wanted, pushed back on what came out, and it wrote the code. I did not hand-write the pitch detection, and I could not have -- I do not know this field.

It works, and there are now 724 tests, none of which need a microphone or a speaker. But it has had no expert review, and the signal-processing choices were made by a model rather than by someone who does this for a living. I am publishing it because the result was more interesting than I expected, not because I am claiming it is correct.

The parts worth reading below are mostly the places where the model was confidently wrong and a measurement settled it.

## Two Bugs That Became Design Decisions

**Legato humming transcribed as a chromatic run.** Singing is legato: the voice *slides* between notes instead of jumping between them. Snapping every analysis frame to the nearest semitone invents a note for each semitone the slide crosses, so humming C-D-E in one breath came back as `C4 C#4 D4 D#4 E4`. That single bug accounted for most of "it never matches what I hummed".

The fix is a glide gate -- discard frames where pitch is moving faster than about 5 semitones per second, keep only held pitch. Getting there needed one step I would never have guessed at. Measuring the slope directly *splits steady notes in two*, because vibrato has a higher instantaneous slope than a glide does: ±40 cents at 5Hz swings past 6 semitones per second. Measuring over a longer window fixes vibrato but then swallows real note changes as well, deleting back-to-back notes entirely.

What works is to median-filter over one vibrato cycle first, then measure a short slope on top. A median removes oscillation but *preserves edges*, so vibrato flattens out while a genuine note change survives as a sharp step. That property is the whole trick.

**Notes split in two at a rounding boundary.** A real recording of me humming low-high-low produced two distinct failures at once. A held note drifted about a semitone while I sang it and, because each frame was snapped and equal values grouped, it broke at the rounding boundary into `F#3 + F3`. And my two attempts at the same low note landed 45 cents apart, either side of a boundary, so they came out as `A2` and `G#2` -- two different notes for something that was musically one.

Two changes. A note is now decided from the median of a whole held region rather than by rounding frame by frame, and the semitone grid itself is shifted by an estimated tuning offset, the way you calibrate a chromatic tuner before trusting it. That recording now transcribes exactly as `G#2 F3 G#2`.

There is a third piece: pitches are clustered across the *whole* recording rather than only between neighbours. Merging adjacent notes cannot fix low-high-low, because the two lows are not adjacent -- the high one sits between them.

## The Crackle, and Two Wrong Answers

Playback crackled in the app while the exact same audio, played from the saved file, was clean.

First hypothesis: GIL contention, the analysis thread starving the audio path. Plausible, cheap to test, and wrong. Instrumented, the underrun counter reported zero.

Second hypothesis: the resampling in the mix path. That turned out to be a genuine defect, and it got fixed. The crackle stayed exactly where it was.

What settled it was not introspection but a phone recording of the speakers while the app played. The rendered file has zero energy above 4kHz, so *any* high-frequency content in that recording has to be an artefact. The artefacts arrived in bursts reaching 0.49 of total energy, spaced a median 64ms apart, against a 65ms output buffer. One click per buffer period is the signature of a buffer underrun, and the numbers left no room for argument.

The cause was pulling audio through a Python callback. A callback must acquire the GIL to run, so while Textual renders the terminal the callback misses its deadline and the device is handed nothing. So the first hypothesis had the right mechanism and the wrong location -- and it had been invisible to every test, because headless Textual never writes to a real terminal and the load that causes it was never present.

Audio is now *pushed* with blocking writes from a worker thread. PortAudio feeds the device from its own ring buffer in C, which never needs the GIL, and `write()` releases the GIL while it waits, so a stalled UI costs latency rather than clicks. Verified the same way it was diagnosed, by playing through the speakers while hammering the main thread: clicky frames fell from 12-19% to 0.0%, peak artefact from 0.49 to 0.015.

I would not have found this from the code. The code looked fine.

## Calibration: Measure the Voice, Do Not Guess At It

Every threshold in the app started out hand-tuned against one person's voice, which is exactly the thing that should be measured per user instead.

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

![Training: hearing a target note, singing it, and being scored](/images/humm2melody/training.gif)

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

Fourteen versions of smaller things, briefly: per-user profiles that remember your dials, voice, notation and tab. Note editing by keyboard or by clicking a note in any of the three views, with insert, delete and 50-deep undo. A piano keyboard you can play -- click keys and it builds a tune, which means you can compose without humming at all. Recordings stored as FLAC (the analysis master, must stay lossless) and playback as MP3 (regenerable from the note list, so it loses nothing), which took a 2.5s run from 354KB to 60KB. Four note-naming schemes, including German, where H is what English calls B and B means B-flat. A tempo dial that regenerates pitch per note rather than resampling, so slowing a melody down to learn it does not transpose it. And a mix dial for playing the tones over your original hum -- the most direct check there is, since tones that sit inside the hum mean it heard you right and tones that beat against it mean it did not.

![Renaming, loading and deleting saved runs](/images/humm2melody/sessions.gif)

Every run is still saved automatically, including the ones where nothing was detected, because a failed transcription is exactly the thing you want to look at later. Each one keeps the hum, the rendered tones, a JSON manifest, and a CSV of every analysis frame -- time, frequency, confidence, loudness, about 43 rows a second. That last file is the detector's frame-by-frame opinion *before* smoothing threw anything away, and it is what made every diagnosis in this post possible.

## What Vibe-Coding It Was Actually Like

Fifteen versions in, the honest version has not changed much.

The model is good at producing something that works and excellent at explaining why it works, including when it does not. It wrote 724 tests, none needing hardware; the strongest renders notes to audio with the playback code, feeds them back through the detector, and checks the same notes come out. That round trip caught real bugs that looked completely reasonable in the source.

It was also wrong in ways that needed pushing back on. It wrote tests asserting the wrong thing and then "fixed" working code to match them -- one insisted 453Hz should be A4 when it is genuinely 50 cents up and rounds to A#4. Twice it reported a defect in the app that was a quirk of the UI framework. And the two most interesting problems in this post, the crackle and the training window, were both cases where the reasoning was fluent, plausible and wrong, and only a measurement from outside the program settled it. A phone held up to some speakers. Singing at the thing and watching what it said.

That is the pattern, and it is not really about models. When the work can be checked against something real, it gets somewhere. When it cannot, it produces confident nonsense at speed -- and so, in fairness, do I.

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
