# Test & Validation Plan

**Version:** 1.3 (final for now)
**Date:** October 30, 2025
**Depends on:** PRD v1.3, Interaction & State Spec v1.3, Hardware Build Spec v1.3

**How to use:** Print this. Check off boxes. Write notes directly on the page. If anything fails, list it under “Blockers” at the end—those become the next iteration tasks.

---

## A. Preconditions / Bench Setup

* ✅ Latest firmware flashed that implements Spec v1.3 constants:

  * LONG_PRESS_THRESHOLD_MS = **1000 ms**
  * INCIDENT_COOLDOWN_MS = **300000 ms (5 min)**
  * OLED_EVENT_DISPLAY_MS = **2000 ms**
* ✅ Final **Pin Map** for your exact ESP32 board taped to bench.
* ✅ microSD (FAT32) inserted; `events.csv` present (or will be created).
* ✅ Battery charged; device powers from LiPo and USB-C.
* ✅ Quiet room available for audio checks; also do one test with normal household noise.

---

## B. Functional Tests (Buttons, Logging, Feedback)

### B1. Per-button mapping & confirmation (tap)

For each front button: VERBAL, PHYSICAL, PROPERTY, REFUSAL, SELF_HARM, REGULATED, INTERVENE(tap=SUPPORT)

Action (x3 each): short tap.
Expect:

* Haptic: **1 buzz**
* OLED: shows timestamp + behavior (and `SUPPORT` for INTERVENE tap)
* CSV row: correct `behavior`, empty `flag` (except as defined below), `incident_file` per state rules

☐ VERBAL tap → `VERBAL`
☐ PHYSICAL tap → `PHYSICAL`
☐ PROPERTY tap → `PROPERTY`
☐ REFUSAL tap → `REFUSAL`
☐ SELF_HARM tap → `SELF_HARM`
☐ REGULATED tap → `REGULATED`
☐ INTERVENE tap → `ATTEMPT_SUPPORT` (no target)

Notes: __________________________________

### B2. Long-press classification (hold ≥1s)

Action (x3 each): press & hold.

Expect:

* Haptic: **2 buzzes**
* OLED shows flag/label (e.g., `threat`, `severe`, `danger`, `BOUNDARY`)
* CSV row: correct `flag` or variant

☐ VERBAL hold → `VERBAL`, **flag=`threat`**
☐ PHYSICAL hold → `PHYSICAL`, **flag=`severe`**
☐ PROPERTY hold *(if implemented)* → `PROPERTY`, **flag=`severe`**
☐ SELF_HARM hold → `SELF_HARM`, **flag=`danger`**
☐ INTERVENE hold → **`ATTEMPT_BOUNDARY`**

**False long-press guard:**
Action: 5 hard, fast **taps** on each of VERBAL and PHYSICAL.
Expect: **0** misreads as hold (acceptance: **0/10** total).
☐ PASS ☐ FAIL  Misreads: ____/10

Notes: __________________________________

### B3. Target tagging (ME/SIB/OTHER one-shot)

Action:

1. Press **SIB** (no CSV row should be created).
2. Tap **PHYSICAL**.
3. Tap **PHYSICAL** again without re-arming.

Expect:

* First PHYSICAL row: `target=SIB`
* Second PHYSICAL row: `target=""` (auto-cleared)
* Haptic/OLED confirm each event; target never attaches to INTERVENE or REGULATED

☐ Target applies once then clears
☐ No CSV row for target button itself

Repeat for **ME** and **OTHER**.

Notes: __________________________________

---

## C. Incident Lifecycle & REC Behavior

### C1. Start from IDLE (problem behavior)

Action: Ensure IDLE (no REC). Tap **PHYSICAL**.

Expect:

* New incident file created (e.g., `incident_YYYY…wav`)
* State → **INCIDENT_ACTIVE**
* **REC** visible on OLED after the event confirmation
* CSV row: `PHYSICAL`, `incident_file=that filename`

☐ New file created
☐ REC shown
☐ CSV references filename

Notes: __________________________________

### C2. Activity during incident

Action sequence:

1. Press **ME**, hold **VERBAL** (→ `threat`).
2. Tap **INTERVENE** (→ SUPPORT).
3. Tap **REGULATED**.
4. Press **SIB**, tap **PROPERTY**.

Expect:

* All rows share **same `incident_file`**
* **Each event resets cooldown**
* OLED shows REC between events

☐ Same file across events
☐ Cooldown resets each event
☐ REC persists

Notes: __________________________________

### C3. Cooldown & incident end

Action: Stop pressing anything; wait full **5 min** (use a timer).

Expect:

* Audio stops
* CSV appends **`INCIDENT_END`** (same `incident_file`)
* State → **IDLE**, REC clears
* Next problem behavior starts a **new** incident file

☐ INCIDENT_END row present
☐ REC cleared
☐ Next behavior starts new file

Notes: __________________________________

### C4. REGULATED & INTERVENE outside incidents

Action (from **IDLE**):

* Tap **REGULATED**
* Tap **INTERVENE** (SUPPORT), then hold **INTERVENE** (BOUNDARY)

Expect:

* All log rows have `incident_file=""` (do **not** start audio)
* State stays **IDLE**, no REC

☐ REGULATED no-incident logging OK
☐ INTERVENE no-incident logging OK

Notes: __________________________________

---

## D. Data Integrity & Audio

### D1. CSV readability

Action: Power down, remove microSD, open `events.csv` on a laptop.

Expect:

* Plain CSV opens cleanly (comma-separated)
* Behavior/flag/target values are human-readable strings
* Incident filename format consistent: `incident_YYYY-MM-DDThh-mm-ss.wav`
* Order matches the event sequence pressed

☐ Readable
☐ Fields correct
☐ Filename format correct

Notes: __________________________________

### D2. Audio integrity

Action: Play the last `incident_*.wav`.

Expect:

* No header corruption; plays end-to-end
* Audio content roughly aligns with CSV timeline (e.g., an apology near REGULATED)
* Motor buzz not overwhelming; speech intelligible (acceptable low-rate quality)

☐ Plays cleanly
☐ Aligns with CSV
☐ Intelligible speech

Notes: __________________________________

### D3. Power loss during incident (safety)

Action: Start incident (tap PHYSICAL), then abruptly cut power (switch/EN). Reboot.

Expect:

* CSV rows up to cut are preserved
* Audio file exists but may end abruptly (acceptable)
* Device boots **IDLE** (no phantom incident)
* `pending_target` cleared

☐ CSV preserved
☐ Audio present
☐ Boot to IDLE
☐ Target cleared

Notes: __________________________________

---

## E. Usability Under Stress

### E1. Eyes-off target + press

Action: Wear on lanyard; without looking, thumb **OTHER** and tap **VERBAL**.

Expect:

* Target sets correctly (OTHER) on that single event
* You can perform in one hand while moving

☐ Target correct
☐ One-handed feasible

Notes: __________________________________

### E2. Rapid multi-press

Action: Tap **VERBAL** 3× in <2 s.

Expect:

* 3 distinct CSV rows
* No freezing; feedback for each

☐ 3 rows present
☐ No freeze

Notes: __________________________________

### E3. OLED glanceability

Action: While walking, press **PHYSICAL**; glance quickly.

Expect:

* You can read behavior/flag/target confirm in ~2 s window
* REC obvious when active

☐ Readable confirm
☐ REC obvious

Notes: __________________________________

---

## F. Survivability & Appearance

### F1. Drop test

Action: With battery connected, drop from chest height onto carpet/couch; pick up and tap **PHYSICAL**.

Expect:

* Device still functions; logs correctly; no reboot loop

☐ Survived; still logs

Notes: __________________________________

### F2. Lanyard yank

Action: Gentle but firm yank on lanyard; then tap **VERBAL**.

Expect:

* No internal disconnect; logs correctly

☐ Survived yank

Notes: __________________________________

### F3. Non-escalating profile

Action: Observe in mirror / with family member present.

Expect:

* Looks neutral; no flashing LEDs; REC only on OLED; not “weapon-like”

☐ Appearance acceptable

Notes: __________________________________

---

## G. Acceptance Thresholds (hard pass/fail)

* **Missed presses:** **0** missed events across all B1–B3 trials.
* **False holds:** **0/10** misreads in B2 guard test (fast hard taps).
* **Incident integrity:** All events during C2 share the same `incident_file`; proper **INCIDENT_END** after cooldown.
* **Audio integrity:** All tested `incident_*.wav` files play cleanly; align with CSV.
* **REC indicator:** Always accurate during ACTIVE/COOLDOWN; off in IDLE.
* **Survivability:** Pass F1 and F2 with device still usable (no reflash).
* **Data readability:** CSV is human-readable; filenames consistent.

Anything less = **NO-GO** for live use.

---

## H. Scenario Run (End-to-End)

Simulated sequence:

1. From IDLE: **PHYSICAL** (SIB) → **VERBAL (threat)** → **ATTEMPT_SUPPORT** → **PROPERTY** → **REGULATED** → **ATTEMPT_BOUNDARY** → (no events 5 min) → auto **INCIDENT_END**.

Verify:

* One audio file
* Correct order in CSV with targets/flags
* REC present during incident; off after
* File plays; sequence audible

☐ PASS  ☐ FAIL  Notes: ___________________________

---

## I. Blockers (if any)

List anything that failed acceptance and what you observed.

1. ---
2. ---
3. ---

---

## J. Go / No-Go

* **GO** if all acceptance thresholds met and Scenario Run passes.
* **NO-GO** if any blocker remains. Document fixes required.

☐ GO   ☐ NO-GO
Sign/Date: _______________________________

