# Code Bluey: Cardiac Rhythm Trainer
## Game Design and Backend Specification (for instructor review)

**Version:** 1.1 · **Audience:** Beginner nursing students (first exposure to ECGs) · **Format:** Browser-based single-page game, no installation, no accounts

---

## What changed since v1.0

1. **Manipulable calipers added.** Students now have a draggable two-legged caliper tool with a live numerical readout, available on the main strip (toggle) and always present on the zoomed measurement views.
2. **Box counting removed.** All instruction, answer choices, feedback, and readouts now use time measurements in seconds. The ECG grid is still drawn (it is what real paper looks like) but is never referenced as a counting method.
3. **Measurement answers are no longer shown.** The zoomed views for the PR and QRS steps previously displayed a bracket marking the interval and its value; this gave the answer away. It has been removed, and students must measure with the caliper.
4. **Findings summary on the identification step.** The final "Name it!" step now displays the strip's verified findings (rate, regularity, P waves, PR, QRS) below the strip so students reason from findings to diagnosis.
5. The regularity step's automatic R-wave overlay was renamed "Auto-march R waves" to distinguish it from the manual calipers.

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

## 4. Measurement tools: the calipers

**Main strip caliper (toggled).** A "Calipers" button under the strip is available on every step. When enabled, a two-legged caliper appears over the strip. Either leg can be dragged independently, or the connecting bar can be dragged to slide the whole caliper along the strip (the "marching out" motion). On the main strip, a dragged leg gently snaps to a nearby R wave, which keeps R-to-R measuring practical on touchscreens.

**Zoomed-view caliper (always present).** During the PR and QRS steps, a second, independent caliper sits on the zoomed view automatically, because measuring is the task on those steps. It starts in a neutral position that does not align with any waveform landmark.

**Readout.** As the student drags, a live readout displays the spanned interval in seconds to two decimals (for example, "0.16 s"). Nothing else is shown: no box conversions and no inferred heart rate.

**Behavior.** Caliper positions reset with each new patient; the on/off state of the main-strip toggle persists across rounds. The calipers are a free measuring instrument and are never graded directly; grading happens only through the step answers.

---

## 5. Round structure and turns

A standard round has 6 steps, completed in fixed order. The student gets exactly one attempt per step; there are no retries. After answering, the game reveals correct or incorrect, shows a short teaching explanation, and the student advances. Wrong answers never end the round.

**Step 1 · Rate (interactive measurement).** The strip is exactly 6 seconds, marked with 3-second indicators. The student taps each R wave; taps snap to the nearest true R peak, and tapping a marked peak unmarks it. The student locks in the count, and the game applies the 6-second method (count × 10). Correct means every R wave was found, no more and no fewer. Missed R waves are then circled in red. A separate "I see no R waves (flatline)" button exists for asystole.

**Step 2 · Regularity (multiple choice, 2 options).** Regular vs. irregular. An optional "Auto-march R waves" toggle overlays arcs between consecutive R waves labeled with each R-R interval in seconds; students can also measure manually with the calipers.

**Step 3 · P waves (multiple choice, 2 options).** One P before every QRS vs. absent or wavy baseline.

**Step 4 · PR interval (multiple choice, 3 options).** Normal (0.12–0.20 s), prolonged (over 0.20 s), or can't measure. A ×3 zoomed view of a single beat is shown with the caliper ready; the student measures from P onset to QRS onset themselves. No part of the answer is drawn on the strip.

**Step 5 · QRS width (multiple choice, 2 options).** Narrow (under 0.12 s) vs. wide (0.12 s or more). Same zoomed view and caliper; the student measures across the QRS themselves.

**Step 6 · Identification (multiple choice, 7 options).** A "Findings from your analysis" panel appears below the strip listing the verified findings for this specific strip: approximate rate in bpm, regularity, P-wave status, and the actual PR and QRS durations in seconds (or "Not measurable"/"n/a" where appropriate). All seven rhythm names are listed as choices. After answering, the student sees a clinical teaching summary for the correct rhythm. Because each earlier step already revealed its correct answer in feedback, the findings panel consolidates known information rather than leaking anything new.

**Asystole exception.** Confirming "no R waves" in Step 1 skips Steps 2 through 5 (unmeasurable on a flatline) and jumps directly to identification, where the findings panel shows "0 bpm (flatline)" and n/a entries. This is intentional teaching.

---

## 6. Scoring

Every correct step earns 10 points on the first (and only) attempt; incorrect earns 0. There is no partial credit and no penalty beyond missed points.

- Standard round: maximum 60 points (6 steps × 10)
- Asystole round: maximum 20 points (rate + identification only)

The round summary shows this round's score out of its maximum, cumulative total score, and total patients seen. The mascot's reaction scales with performance. A count of perfect rounds is tracked internally but not displayed.

---

## 7. ECG rendering specifications

- Strip length: 6.0 seconds at a simulated 25 mm/s paper speed
- Grid: standard ECG paper proportions (1 small division = 0.04 s, heavy line every 0.20 s); classic pale-red grid on off-white. The grid is visual context only; no gameplay text references counting its divisions.
- Waveform: sampled at 150 points per second and drawn as a continuous SVG path; animates left-to-right on each new strip (disabled automatically for users with a reduced-motion system setting)
- Beat morphology: P wave, QRS, and T wave built from piecewise-linear templates; V-tach uses a wide slurred template; A-fib adds a low-amplitude multi-frequency wobble as the fibrillatory baseline; asystole is a slow low-amplitude drift
- Zoomed view (Steps 4–5): horizontal time scale is exactly ×3 and accurate for caliper measurement; it carries no measurement markings. One honest limitation: the vertical (amplitude) scale is compressed slightly to fit the panel. Since only horizontal (time) measurements are ever asked, this affects no question, but the view is not vertically calibrated.

---

## 8. Data, privacy, and technical requirements

The game stores nothing. There are no accounts, no server, no cookies, and no persistent storage of any kind; refreshing the page resets the score to zero. There is no instructor dashboard and no way to verify that a student played or how they scored. It is built for ungraded practice; graded or tracked use would require a different architecture.

Runs in any modern browser on desktop, tablet, or phone. Touch, mouse, and keyboard focus are supported; caliper dragging uses pointer events, which cover both mouse and touch. No internet connection is needed after the page loads, except for two decorative web fonts that fall back to system fonts if unavailable.

**Builds.** The deployable build is the standalone single-file `code-bluey.html` (canonical, matches this spec). An earlier React artifact version exists but reflects v1.0 only and does not include the calipers, the seconds-based text, or the findings panel.

---

## 9. Known limitations for instructor awareness

1. Strips are idealized; students trained only on these will find real telemetry messier.
2. Rate checking requires finding every R wave exactly; there is no ±1 tolerance. Defensible for beginners, but easy to relax if preferred.
3. Caliper drag precision on small phone screens has not been validated on physical devices; the snap-to-R assist and the ×3 zoom mitigate this, but it should be tried on the devices students will actually use before rollout.
4. Regularity is answered categorically; students are not asked to quantify R-R variance.
5. The identification step always lists all 7 options in the same order, which savvy students may pattern-match; options could be shuffled.
6. No time pressure anywhere; there is no timed mode yet.
7. Amplitude and voltage criteria (axis, hypertrophy, ST analysis) are entirely out of scope; appropriate for a rhythm-strip course, but worth stating.

---

*Version 1.1. All specifications above describe the current build of code-bluey.html exactly as implemented and tested.*
