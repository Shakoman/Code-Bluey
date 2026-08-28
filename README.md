# Code Bluey: Cardiac Rhythm Trainer
## Game Design and Backend Specification (for instructor review)

**Version:** 1.0 draft · **Audience:** Beginner nursing students (first exposure to ECGs) · **Format:** Browser-based single-page game, no installation, no accounts

---

## 1. Overview

Code Bluey teaches the five-step systematic method of rhythm strip analysis. Each round presents one simulated "patient" with a computer-generated 6-second ECG strip. The student works through the analysis steps in a fixed order, receives immediate feedback after every answer, and finishes by naming the rhythm. A cartoon mascot ("Code Bluey," an original blue dog character) delivers instructions and coaching in a speech bubble.

All strips are generated programmatically as SVG waveforms. They are idealized teaching approximations, not real patient tracings; they contain no artifact, baseline wander, or lead noise. A disclaimer stating this appears on the start screen.

---

## 2. Patients and session length

The game is endless. Each round is one patient, and a new patient can be started indefinitely by pressing "Next patient." There is no fixed number of patients, no lives system, and no game-over state. A session ends whenever the student closes the page.

The rhythm for each patient is selected at random from the rhythm library, with one constraint: the same rhythm never appears twice in a row. Beyond that, selection is uniform, so over a long session each rhythm appears roughly one-seventh of the time. There is currently no difficulty progression and no weighting toward rhythms the student has missed.

If you would prefer a fixed-length quiz mode (for example, 10 patients and a final grade), that is a straightforward change.

---

## 3. Rhythm library (7 rhythms)

Each rhythm is generated fresh every time it appears; rates and (for A-fib) beat spacing are randomized within the ranges below, so no two strips are identical.

| Rhythm | Generated rate | Regularity | P waves | PR interval | QRS |
|---|---|---|---|---|---|
| Normal sinus rhythm | 64–92 bpm | Regular | Present | 0.16 s (normal) | 0.08 s (narrow) |
| Sinus bradycardia | 40–55 bpm | Regular | Present | 0.17 s (normal) | 0.08 s (narrow) |
| Sinus tachycardia | 105–122 bpm | Regular | Present | 0.14 s (normal) | 0.08 s (narrow) |
| 1st-degree AV block | 58–82 bpm | Regular | Present | 0.28 s (prolonged) | 0.08 s (narrow) |
| Atrial fibrillation | Variable (R-R randomized 0.48–0.92 s, roughly 65–125 bpm) | Irregularly irregular | Absent (fibrillatory baseline drawn instead) | Not measurable | 0.08 s (narrow) |
| Ventricular tachycardia | 150–175 bpm | Regular | Absent | Not measurable | 0.16 s (wide) |
| Asystole | 0 | n/a | n/a | n/a | n/a |

Deliberately excluded from the beginner set: 2nd-degree blocks (Type I and II), 3rd-degree block, SVT, atrial flutter, PVCs/PACs, ventricular fibrillation, and paced rhythms. Any of these can be added later.

---

## 4. Round structure and turns

A standard round has 6 steps, completed in fixed order. The student gets exactly one attempt per step; there are no retries. After answering, the game reveals correct/incorrect, shows a short teaching explanation, and the student advances to the next step. Wrong answers never end the round; the student always completes all steps and names the rhythm.

**Step 1 · Rate (interactive measurement).** The strip is exactly 6 seconds, marked with 3-second indicators. The student taps each R wave; taps snap to the nearest true R peak (generous tolerance for touchscreens), and tapping a marked peak unmarks it. The student locks in the count, and the game applies the 6-second method (count × 10). Correct means every R wave was found, no more and no fewer. Missed R waves are then circled in red on the strip. A separate "I see no R waves (flatline)" button exists for asystole.

**Step 2 · Regularity (multiple choice, 2 options).** Regular vs. irregular. An optional "Show calipers" toggle overlays arcs between consecutive R waves labeled with each R-R interval in seconds, teaching students to march out the rhythm.

**Step 3 · P waves (multiple choice, 2 options).** One P before every QRS vs. absent/wavy baseline.

**Step 4 · PR interval (multiple choice, 3 options).** Normal (3–5 small boxes), prolonged (more than 5), or can't measure. A ×3 zoomed view of a single beat is shown with a bracket marking exactly the span to measure and the small-box count.

**Step 5 · QRS width (multiple choice, 2 options).** Narrow (under 3 small boxes) vs. wide (3 or more). Same zoomed view, bracket now marks the QRS.

**Step 6 · Identification (multiple choice, 7 options).** All seven rhythm names are always listed. After answering, the student sees a clinical teaching summary for the correct rhythm (for example, for V-tach: check the patient immediately; pulse vs. no pulse changes everything).

**Asystole exception.** When the strip is asystole, confirming "no R waves" in Step 1 skips Steps 2 through 5 (they are unmeasurable on a flatline) and jumps directly to identification. This is intentional teaching: recognizing that a flatline has nothing to measure is the lesson.

---

## 5. Scoring

Every correct step earns 10 points on the first (and only) attempt; incorrect earns 0. There is no partial credit and no penalty for wrong answers beyond the missed points.

- Standard round: maximum 60 points (6 steps × 10)
- Asystole round: maximum 20 points (rate + identification only)

The round summary screen shows: this round's score out of its maximum, cumulative total score, and total patients seen. The mascot's reaction scales with performance (celebration for a perfect round, encouragement otherwise). The game also tracks a count of perfect rounds internally, though this is not currently displayed anywhere; I can surface it or remove it.

---

## 6. ECG rendering specifications

- Strip length: 6.0 seconds at a simulated 25 mm/s paper speed
- Grid: standard ECG paper proportions; 1 small box = 0.04 s, heavy line every 5 boxes (0.20 s); classic pale-red grid on off-white
- Waveform: sampled at 150 points per second and drawn as a continuous SVG path; animates left-to-right on each new strip (animation is disabled automatically for users with a reduced-motion system setting)
- Beat morphology: P wave, QRS, and T wave built from piecewise-linear templates; V-tach uses a wide slurred template; A-fib adds a low-amplitude multi-frequency wobble as the fibrillatory baseline; asystole is a slow low-amplitude drift
- Zoomed view (Steps 4–5): horizontal time scale is exactly ×3 and accurate for box counting. One honest limitation: the vertical (amplitude) scale in the zoomed view is compressed slightly to fit the panel, so vertical box counts would not be accurate there. Since the zoom is only used for horizontal (time) measurements, this does not affect any question, but you should know it is not vertically calibrated.

---

## 7. Data, privacy, and technical requirements

The game stores nothing. There are no accounts, no server, no cookies, and no persistent storage of any kind; refreshing the page resets the score to zero. Consequently there is no instructor dashboard and no way to verify that a student played or how they scored. It is built for ungraded practice. If you later want graded or tracked use, that requires a different architecture, and I can outline options.

Runs in any modern browser on desktop, tablet, or phone. Touch, mouse, and keyboard focus are all supported. No internet connection is needed after the page loads, except for two decorative web fonts that fall back to system fonts if unavailable.

---

## 8. Known limitations for instructor awareness

1. Strips are idealized; students trained only on these will find real telemetry messier.
2. Rate checking requires finding every R wave exactly; there is no ±1 tolerance. For beginners this is defensible (it builds care) but it can feel strict; easy to relax if you prefer.
3. Regularity is answered categorically; students are not asked to measure R-R variance numerically.
4. The identification step always lists all 7 options in the same order, which savvy students may pattern-match; options could be shuffled.
5. No time pressure anywhere; there is no timed mode yet.
6. Amplitude/voltage criteria (axis, hypertrophy, ST analysis) are entirely out of scope, appropriate for a rhythm-strip course but worth stating.

---

*Prepared for review prior to deployment. All specifications above describe the current build (v1.0 draft) exactly as implemented.*
