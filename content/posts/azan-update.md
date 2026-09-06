---
title: "Azan 0.2.3: The Hour That Wasn't There"
date: 2026-09-06
draft: false
tags: ["garmin", "prayer-times", "connect-iq", "testing"]
summary: "Prayer times for any date but today could be a full hour wrong. The bug had been there since the beginning, and the tests that would have caught it did not exist."
cover:
  image: "/images/azan-update/02_prayer_time.jpg"
  alt: "Azan prayer times on a Garmin Enduro 3"
---

Azan sits at 3.17 stars on the Connect IQ store. The reviews are consistent
about what is wrong, and they have been for months: *"prayer times are off"*,
*"the time is wrong"*, *"off by 5 to 30 minutes"*.

I had read those as a calculation-method problem. Some of them are. One of them
was not, and it was mine.

## An hour, not five minutes

Ask Azan for prayer times on a date that is not today — scroll the calendar to
December while it is September — and it calculated them with **today's**
timezone offset.

`System.getClockTime()` reports the offset in effect right now. It has no notion
of the date you asked about. So a December date computed in September used
central European summer time instead of winter time, and every prayer came out
an hour late. Fajr, Dhuhr, Asr, Maghrib, Isha: all of them, all wrong, by
exactly sixty minutes.

That is not a subtle astronomical disagreement. That is the app confidently
displaying the wrong hour.

It was worse than a calendar curiosity. The glance — the strip you see before
opening the widget — computes yesterday and tomorrow to work out the next
prayer countdown. Near a daylight-saving changeover, one of those two dates sits
on the far side of the boundary. So the countdown on the watch face could be
wrong without anyone touching the calendar at all.

## The fix

The offset has to come from the date being asked about, not from the clock.

Connect IQ has no "what was the offset on this date" call, but it has the
pieces. `Gregorian.moment` treats its components as UTC. So: take noon UTC on
the target date, read it back as local components, re-encode those as UTC, and
subtract. The difference is the offset that applied on that date.

The tempting shortcut is to compare hour numbers and wrap at 24. Do not. That
approach quietly breaks for half-hour zones like India, three-quarter-hour zones
like Nepal, and anything past UTC+12 like Kiribati — users far enough away that
they would never be able to describe the failure coherently. Going through
moment values avoids the arithmetic entirely.

Verified against the real 2026 European boundaries. 29 March returns +2,
25 October returns +1. Those are the actual switch dates.

## The other two

**The app could wait forever.** `Position.enableLocationEvents` can be accepted
by the platform and then never answered — indoors, or with GPS unavailable.
Nothing cleared the waiting state, so the widget showed "Acquiring location"
indefinitely. There is now a thirty-second timeout.

The interesting part: the app already had a perfectly good "Enable GPS — to
calculate prayer times" message. It had never once been displayed. The branch
above it always won, because the loading flag never became false without a
callback. Adding the timeout did not add that message; it made four-month-old
dead code reachable for the first time.

**The sun chart had two overlapping defects, on different axes.** The title was
painted over by the coloured prayer bands, which start after it and reached
above its baseline — "Sun Position" was cut in half. Separately, on the 176-pixel
Instinct 2 the evening bands are only a few pixels wide, and the Asr and Maghrib
letters landed on top of each other.

Only the second was visible to the test suite, which compares text against text.
The first was text against graphics, and nothing could see it. I found it by
taking a screenshot for the store listing and looking at it.

## What the tests could and could not do

The suite went from 20 tests to 37, passing on Instinct 2, Fenix 5X, Fenix 7S,
Venu 3 and Enduro 3, with all 121 declared devices compiling.

Both real bugs shared a shape. The daylight-saving bug hid behind
`System.getClockTime()`, which no test could vary. The sun-chart title hid in a
drawing routine that needed a screen. Neither was reachable until the logic was
pulled out into a plain function taking explicit arguments — an offset for a
date, a band top for a title height. After that they were trivial to test.

A test suite that has never been seen to fail proves nothing, so each fix was
checked by reintroducing the bug on purpose and confirming the right tests went
red. The daylight-saving ones fail without the fix. The band-geometry one fails
if the old maths comes back.

## What is still wrong

The reviews that say *five to thirty minutes* are not explained by any of this.

Methods like Diyanet and Umm al-Qura do not publish a formula. They publish
per-city timetables containing local adjustments that no astronomical model
reproduces, because they are not astronomical — they are decisions. You can
approximate them. You cannot derive them. I spent real effort trying to fit a
model to Diyanet's published times and the honest conclusion is that there is no
function to find.

So a few minutes of divergence from your local authority can remain, and the
store description now says so rather than leaving people to discover it. If your
mosque follows a different method, switching it in settings is the fix.

Two other things are deliberately unaddressed: compass calibration against true
north, and modelling the Instinct's semi-octagon screen shape. Both are real.
Neither is worth the maintenance budget right now.

## Get it

Search for **Azan** in the Connect IQ Store, or at
[apps.garmin.com](https://apps.garmin.com/).

If you ever scrolled the calendar to a date months away and thought the times
looked strange: you were right, and it took me until now to notice.
