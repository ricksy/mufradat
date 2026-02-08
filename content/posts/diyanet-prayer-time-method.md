---
title: "The Diyanet Prayer Time Method: A Black Box That Can't Be Reproduced"
date: 2026-02-05
draft: false
tags: ["prayer-times", "diyanet", "open-source", "islam", "astronomy"]
summary: "After months of trying to reproduce the Turkish Diyanet prayer time calculations, we found systematic discrepancies that no amount of parameter tuning can explain. Here's what we found -- and why it matters."
cover:
  image: "/images/diyanet/berlin-differences.png"
  alt: "Chart showing prayer time differences between calculated and Diyanet reference data for Berlin"
  caption: "Fajr and Isha differences spike in April and August across all European cities we tested."
---

## Background

While building [Azan]({{< ref "/posts/azan-garmin-widget" >}}), a prayer time app for Garmin watches, I needed to implement the Turkish Diyanet calculation method. Diyanet (the Presidency of Religious Affairs in Turkey) is one of the most widely used prayer time authorities in Europe, serving millions of Muslims who rely on its published times for their daily prayers.

Unlike most other calculation methods -- Muslim World League, ISNA, Egyptian General Authority -- the Diyanet method is not open source. There is no published paper, no documented algorithm, no reference implementation. What exists is a website ([namazvakitleri.diyanet.gov.tr](https://namazvakitleri.diyanet.gov.tr)) that outputs times, and third-party APIs that relay those times. The algorithm behind them is a black box.

This post documents our attempt to reverse-engineer the Diyanet method, what we found, and why we believe the method cannot be faithfully reproduced through pure astronomical calculation.

## What We Know About the Diyanet Parameters

From third-party sources and community documentation, the Diyanet method is understood to use:

- **Fajr angle**: 18.0° (sun depression below horizon)
- **Isha angle**: 17.0°
- **Time adjustments**: Small offsets applied to each prayer (e.g., Sunrise -7 min, Dhuhr +5 min, Maghrib +7 min)

These parameters are what other prayer time libraries (like the widely-used [Adhan](https://github.com/batoulapps/adhan-js) library and the [AlAdhan API](https://aladhan.com/prayer-times-api)) use when you select "Diyanet" or "Turkey" as your method.

We implemented these parameters faithfully and validated them against 11 European cities over a full year.

## Our Methodology

We built a prayer time calculation engine in both [Monkey C](https://developer.garmin.com/connect-iq/) (for the Garmin app) and Python (for validation), using the same astronomical algorithms as the Adhan library:

1. **Solar position**: Julian date, equation of time, solar declination using standard VSOP-based formulas
2. **Hour angle calculation**: Standard formula for sun angle at a given depression
3. **High-latitude adjustment**: Twilight angle rule (fraction of night based on angle/divisor)
4. **Reference data**: Fetched from the AlAdhan API (method 13 = Diyanet) for all cities, full year 2026

We then compared our calculated times against the Diyanet reference for **11 European cities**:

| City | Latitude |
|------|----------|
| Oslo | 59.91°N |
| Stockholm | 59.33°N |
| Berlin | 52.52°N |
| Amsterdam | 52.37°N |
| Rotterdam | 51.92°N |
| London | 51.51°N |
| Brussels | 50.85°N |
| Paris | 48.86°N |
| Vienna | 48.21°N |
| Munich | 48.14°N |
| Zurich | 47.38°N |

## The Results: Something Doesn't Add Up

For most prayers and most months, our implementation tracks the Diyanet reference data closely. Dhuhr, Asr, Sunrise, and Maghrib average less than 1 minute of error year-round. These are prayers determined by straightforward solar geometry -- noon, shadow length, horizon crossing -- and they confirm our astronomical calculations are sound.

But **Fajr and Isha tell a different story**.

### Berlin (52.52°N) -- Full Year Breakdown

| Month | Fajr Error | Isha Error | Total Daily Error |
|-------|-----------|-----------|-------------------|
| January | 0.8 min | 0.9 min | 3.81 min |
| February | 1.4 min | 0.7 min | 4.11 min |
| March | 1.3 min | 0.9 min | 4.10 min |
| **April** | **3.6 min** | **1.7 min** | **7.07 min** |
| **May** | **2.9 min** | **3.1 min** | **7.58 min** |
| June | 2.4 min | 0.6 min | 4.70 min |
| July | 1.5 min | 2.2 min | 5.48 min |
| **August** | **3.9 min** | **2.7 min** | **8.71 min** |
| September | 0.0 min | 0.3 min | 2.87 min |
| October | 0.2 min | 0.6 min | 3.16 min |
| November | 0.2 min | 0.2 min | 2.60 min |
| December | 0.5 min | 0.3 min | 2.94 min |

The pattern is unmistakable: errors concentrate in **April-May** and **July-August**, precisely the months when twilight duration changes most rapidly at northern European latitudes. Winter months are nearly perfect. The transition periods are where the Diyanet method diverges from what any consistent set of angles and parameters can produce.

### The Fajr Chart -- Berlin

![Fajr prayer time comparison for Berlin](/images/diyanet/berlin-fajr.png)

The blue line is the Diyanet reference. The red dashed line is our calculation. The green shaded area shows the difference. Notice how the lines track closely in winter, then diverge sharply in spring and late summer. Our calculated Fajr is consistently *later* than the Diyanet reference during these periods -- by up to 10+ minutes in some days.

### The Isha Chart -- Berlin

![Isha prayer time comparison for Berlin](/images/diyanet/berlin-isha.png)

The same pattern appears for Isha, with the Diyanet reference being *later* than our calculation in the same problematic months.

### The Difference Chart -- All Prayers, Berlin

![All prayer time differences for Berlin](/images/diyanet/berlin-all-differences.png)

This chart shows all six prayers on one plot. Dhuhr, Asr, Sunrise, and Maghrib stay flat near zero all year. Fajr and Isha are the outliers, with dramatic seasonal swings that no single set of parameters can explain.

## The Pattern Holds Across All Cities

This is not a Berlin-specific anomaly. Every city we tested shows the same signature: stable agreement in winter, divergence in the spring and late summer transition months, with the magnitude scaling with latitude.

### Stockholm (59.33°N)

![Prayer time differences for Stockholm](/images/diyanet/stockholm-differences.png)

At Stockholm's higher latitude, the Fajr discrepancies reach even larger values during the problematic months.

### Oslo (59.91°N)

![Prayer time differences for Oslo](/images/diyanet/oslo-differences.png)

Oslo, at nearly 60°N, shows the most extreme divergence. The Fajr error peaks exceed 15 minutes in April.

### London (51.51°N)

![Prayer time differences for London](/images/diyanet/london-differences.png)

Even London, at a more moderate latitude, shows the same April/August pattern, just at a smaller scale.

## What We Tried

We did not give up easily. We wrote multiple experimental scripts to try every reasonable approach to match the Diyanet output:

### 1. Angle Tuning
We reverse-engineered the implied sun depression angles from the Diyanet times for every day of the year. If Diyanet used a consistent angle, we should find a stable value. Instead, we found the implied Fajr angle **varies seasonally** -- it is not a fixed 18°. The implied angle drifts between roughly 14° and 22° depending on the month and latitude, which is physically meaningless for a method supposedly based on a fixed twilight angle.

### 2. High-Latitude Divisor Tuning
The standard twilight angle rule uses `angle / 60` as the fraction of night. We swept divisors from 75 to 90 and found that 86-87.5 gives a better overall balance. But no single divisor eliminates the April/August discrepancy -- it just shifts the error between months.

### 3. Night Fraction Caps
We tested explicit caps on Fajr and Isha as fractions of the night length (1/5, 1/4.5, various values from 0.18 to 0.25). Some configurations reduce the error in the problematic months but increase it elsewhere. There is no single pair of fractions that matches the Diyanet output year-round.

### 4. Equation of Time Variants
We tested both the standard equation of time (using mean longitude L0) and an apparent longitude variant. The difference is small and doesn't explain the seasonal pattern.

### 5. Combination Approaches
We tested nine different configurations combining all of the above, evaluating each against both the problematic months (April/August) and the full year. The best we achieved was an average of ~1.15 minutes per prayer across 11 cities -- good, but with irreducible spikes in the transition months that no parameter combination can flatten.

## Our Conclusion: It's Probably a Lookup Table

After exhausting every calculable approach, we reached a conclusion: **the Diyanet method almost certainly does not rely purely on astronomical calculation**.

The evidence points to a system that uses either:

- **Lookup tables** with manually curated adjustments for specific latitude bands and seasons
- **Interpolation** between pre-computed reference points that don't follow a single analytical formula
- **Manual corrections** applied by religious scholars for specific periods (possibly related to when astronomical twilight becomes ambiguous at high latitudes)

The seasonal pattern of the discrepancies -- concentrated exactly in the months when twilight angle calculations become sensitive at northern latitudes -- strongly suggests human intervention in the output. This is not necessarily wrong from a religious perspective. But it means the Diyanet method **cannot be faithfully implemented by any third-party developer using only the published parameters**.

## Why This Matters

### For App Developers
Every prayer time app that claims to support the "Diyanet method" is, at best, an approximation. The parameters (18°/17° angles, minute adjustments) give you close results for most of the year, but they will diverge from the actual Diyanet website output by several minutes during critical transition months. Developers cannot fix this because the actual method is not documented.

### For Muslims Relying on These Apps
If you use a prayer time app set to "Diyanet" and live in Northern Europe, your Fajr time during April or August may differ from the official Diyanet website by 5-10 minutes. During Ramadan, when Fajr determines the start of fasting, this difference is not trivial.

### For the Broader Community
Prayer time calculation is a solved problem in astronomy. The sun's position can be calculated to sub-second accuracy. What varies between methods is the **choice of parameters** -- what sun angle defines Fajr, what safety margins to apply. These are legitimate scholarly decisions. But they should be **transparent**.

Other major calculation authorities publish their parameters openly:
- **Muslim World League**: Fajr 18°, Isha 17° (documented)
- **ISNA**: Fajr 15°, Isha 15° (documented)
- **Egyptian General Authority**: Fajr 19.5°, Isha 17.5° (documented)
- **Umm Al-Qura**: Fajr 18.5°, Isha 90 min after Maghrib (documented)

The Diyanet method's parameters can only be inferred through reverse engineering, and as we've shown, those inferred parameters don't fully explain the output. This opacity is a disservice to the millions of Muslims who depend on accurate prayer times.

## A Call for Transparency

We are not questioning the religious authority of Diyanet or the validity of their prayer times. The times they publish may well incorporate sound scholarly reasoning for the adjustments they make.

What we are asking for is simple: **publish the algorithm**. Document the parameters, the high-latitude rules, the seasonal adjustments, whatever lookup tables or correction factors are used. Let developers implement it correctly. Let the community verify it. Let scholars from other traditions understand and evaluate the methodology.

Prayer times are too important to be a black box.

---

## Technical Details

The analysis was performed using a Python calculation engine and validation suite built as part of [Azan]({{< ref "/posts/azan-garmin-widget" >}}), a Garmin prayer time app. The key scripts used in this research:

- **Prayer time engine** -- Standard astronomical algorithms (Julian date, solar position, hour angle) aligned with the [Adhan](https://github.com/batoulapps/adhan-js) library
- **Reverse engineering** -- Calculated implied sun depression angles from observed Diyanet times for every day of the year across 11 cities
- **Parameter tuning** -- Swept high-latitude divisors (75--90), night fractions (0.18--0.25), and equation of time variants
- **Multi-city validation** -- Compared calculated vs reference times for 11 European cities (47°N--60°N) over a full year

Reference data was fetched from the [AlAdhan API](https://aladhan.com/prayer-times-api) using method 13 (Diyanet/Turkey). If you're interested in the code or data, feel free to [get in touch](/about/).
