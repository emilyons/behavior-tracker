# Interaction & State Spec

**Version:** 1.3
**Date:** October 28, 2025
**Depends on:** PRD v1.3

This document defines how the device behaves at runtime:

* Firmware state machine
* Button behavior (tap vs hold)
* Target tagging logic
* Incident recording logic + cooldown
* CSV logging format and naming
* Haptic + OLED feedback
* Timing constants and robustness expectations

This is the implementation contract. Firmware must match this spec.

---

## 1. Core Concepts

### 1.1 Event

An **event** is anything the caregiver logs via a front-face button (VERBAL, PHYSICAL, etc., including INTERVENE and REGULATED), plus the automatic `INCIDENT_END` row.

Each event:

* Creates a row in `events.csv`
* May update or extend an active incident
* Triggers haptic + OLED confirmation (except `INCIDENT_END`, which is system-generated)

### 1.2 Incident

An **incident** is an escalation window:

* Starts with the first logged *problem behavior* from IDLE
* Keeps going while related events continue
* Ends automatically after a cooldown period with no events
* Is associated with exactly one audio recording file

The incident is how we tie behavior, targets, caregiver response, and audio context together.

### 1.3 Target

A **target** is who the behavior was directed at:

* ME (caregiver)
* SIB (sibling)
* OTHER (teacher, stranger, etc.)
* blank

Targets apply only to the next logged behavior event, and then auto-clear.

### 1.4 Intervention

**Intervention events** (ATTEMPT_SUPPORT / ATTEMPT_BOUNDARY) capture what the caregiver did in response (support vs safety boundary). These are separate from the child’s behavior and must be logged distinctly.

---

## 2. Device States

The firmware must maintain one of three high-level states:

1. **IDLE**

   * No active incident
   * Not recording audio
   * Cooldown timer not running

2. **INCIDENT_ACTIVE**

   * Incident exists
   * Recording audio to current incident file
   * Events are still actively happening
   * Cooldown timer resets after every new event

3. **INCIDENT_COOLDOWN**

   * Same incident is still “open”
   * Audio is still recording
   * We’re waiting to see if the situation restarts
   * Cooldown timer is counting down (e.g. 5 minutes since last event)
   * Any event during cooldown pulls us back into INCIDENT_ACTIVE

No other states are allowed. Firmware uses only these three.

---

## 3. State Transitions

This section defines exactly when the device changes from one state to another.

### 3.1 IDLE → INCIDENT_ACTIVE

Condition:

* The user logs any *problem behavior* while IDLE:

  * VERBAL
  * PHYSICAL
  * PROPERTY
  * REFUSAL
  * SELF_HARM

Actions:

1. Create new incident ID.
2. Generate a new audio filename, e.g. `incident_2025-10-28T20-41-03.wav`.
3. Begin audio recording to that file.
4. Log the event to CSV with `incident_file` set to that filename.
5. Set state = INCIDENT_ACTIVE.
6. Start/reset the cooldown timer.
7. OLED must start showing `REC` state when not in transient “last event” mode.

Notes:

* REGULATED and INTERVENE do **not** create a new incident from IDLE.
* This prevents starting audio just because I tried to help or she got momentarily sweet.

### 3.2 INCIDENT_ACTIVE → INCIDENT_ACTIVE

Condition:

* Any new event is logged while already in INCIDENT_ACTIVE.

Events include:

* VERBAL / PHYSICAL / PROPERTY / REFUSAL / SELF_HARM
* REGULATED
* ATTEMPT_SUPPORT (INTERVENE tap)
* ATTEMPT_BOUNDARY (INTERVENE hold)

Actions:

1. Log the event with the current `incident_file`.
2. Trigger haptic + OLED confirmation.
3. Reset cooldown timer.
4. Stay in INCIDENT_ACTIVE.
5. Keep audio recording.

Notes:

* REGULATED counts as activity.
* INTERVENE counts as activity.
* This means an apology burst or a caregiver boundary still keeps the same incident/audio file alive.

### 3.3 INCIDENT_ACTIVE → INCIDENT_COOLDOWN

Condition:

* The last event just happened, and we are now waiting to see if it’s really over.

Implementation detail:

* The transition to INCIDENT_COOLDOWN can happen after a short idle window (e.g. immediately or a few seconds after last event). The important part is: cooldown timing is now ticking.

Actions:

1. Set state = INCIDENT_COOLDOWN.
2. Start (or continue) the cooldown timer.
3. Keep audio recording to the same file.
4. OLED still shows `REC` during idle periods.

### 3.4 INCIDENT_COOLDOWN → INCIDENT_ACTIVE

Condition:

* Any new event is logged during cooldown.

Actions:

1. Log event with same `incident_file`.
2. Trigger haptic + OLED confirmation.
3. Reset cooldown timer.
4. Set state = INCIDENT_ACTIVE.
5. Continue recording audio to the same file (no split).

### 3.5 INCIDENT_COOLDOWN → IDLE

Condition:

* The cooldown timer expires (default: 5 minutes = 300000 ms) with no new events.

Actions:

1. Stop audio recording.
2. Append `INCIDENT_END` row to CSV:

   * behavior = `INCIDENT_END`
   * same `incident_file`
   * target = ""
   * flag = ""
3. Clear the active incident ID / filename.
4. Clear `REC` indicator on OLED.
5. Set state = IDLE.
6. Reset `pending_target` (target should not persist across incidents).

---

## 4. Button Input Behavior

There are two button classes:

* **Front-face main buttons** (behaviors and caregiver interventions)
* **Side target buttons** (ME / SIB / OTHER)

### 4.1 Front-face main buttons

Front buttons are:

* VERBAL
* PHYSICAL
* PROPERTY
* REFUSAL
* SELF_HARM
* REGULATED
* INTERVENE (special — maps to ATTEMPT_SUPPORT / ATTEMPT_BOUNDARY)

All front buttons support:

* Tap (short press)
* Hold (long press, ≥ LONG_PRESS_THRESHOLD_MS)

Firmware must measure press duration and classify tap vs hold.

#### VERBAL

* Tap:

  * behavior = `VERBAL`
  * flag = `""`
* Hold:

  * behavior = `VERBAL`
  * flag = `threat`
  * Used for explicit violent threats (“I’ll stab you”).

#### PHYSICAL

* Tap:

  * behavior = `PHYSICAL`
  * flag = `""`
* Hold:

  * behavior = `PHYSICAL`
  * flag = `severe`
  * Used for high-risk or injurious hits/kicks/throws.

#### PROPERTY

* Tap:

  * behavior = `PROPERTY`
  * flag = `""`
* Hold (optional in v1 firmware):

  * behavior = `PROPERTY`
  * flag = `severe`
  * For dangerous property destruction. If not implemented in v1, hold == tap.

#### REFUSAL

* Tap:

  * behavior = `REFUSAL`
  * flag = `""`
* Hold:

  * In v1, hold may equal tap (no special flag).

#### SELF_HARM

* Tap:

  * behavior = `SELF_HARM`
  * flag = `""`
* Hold:

  * behavior = `SELF_HARM`
  * flag = `danger`
  * For acute self-harm / high-risk self-endangerment.

#### REGULATED

* Tap:

  * behavior = `REGULATED`
  * flag = `""`
  * Represents calm/apology/sweet repair.
* Hold:

  * Same as tap. REGULATED does not need a long-press variant in v1.

Rules for REGULATED:

* If state = IDLE:

  * Log `REGULATED` with `incident_file=""`.
  * Stay in IDLE.
  * Do NOT start audio.
* If state = INCIDENT_ACTIVE or INCIDENT_COOLDOWN:

  * Log `REGULATED` with the active `incident_file`.
  * Reset cooldown timer.
  * Remain in (or go back to) INCIDENT_ACTIVE.

This captures apology/softening as part of the same escalation, and keeps the same audio file alive.

#### INTERVENE (caregiver action)

INTERVENE is its own physical button.

* Tap:

  * behavior = `ATTEMPT_SUPPORT`
  * flag = `""`
  * Meaning: I attempted supportive/co-regulation strategies (comfort, snack, calm voice, sensory support, etc.).
* Hold:

  * behavior = `ATTEMPT_BOUNDARY`
  * flag = `""`
  * Meaning: I enforced a safety boundary (blocking harm, separating siblings, removing an object, “you can’t hit her”).

Rules for INTERVENE:

* If state = IDLE:

  * Log the row (ATTEMPT_SUPPORT or ATTEMPT_BOUNDARY).
  * `incident_file = ""`.
  * Do NOT start a new incident.
  * Do NOT start audio.
* If state = INCIDENT_ACTIVE or INCIDENT_COOLDOWN:

  * Log the row with the active `incident_file`.
  * Reset cooldown timer.
  * Set / remain in INCIDENT_ACTIVE.
  * Keep audio rolling.

INTERVENE events never take a target (target is always logged as blank).

---

### 4.2 Side target buttons (ME / SIB / OTHER)

Side buttons:

* ME
* SIB
* OTHER

These are momentary. Pressing one:

1. Sets `pending_target` to that value.
2. Does NOT create a CSV row.
3. Does NOT affect incident state by itself.

When the next behavior event (VERBAL, PHYSICAL, PROPERTY, REFUSAL, SELF_HARM, REGULATED) — **not INTERVENE** — is logged:

* That event’s CSV row gets `target = pending_target`.
* After logging, firmware must immediately clear `pending_target = ""`.

If no target is armed, `target` in the CSV row is `""`.

`pending_target` must also be cleared:

* When an incident ends and we transition to IDLE.
* On reboot.

---

## 5. CSV Logging Rules

Each event that logs creates one CSV row, appended to `events.csv`.

Columns:

1. `timestamp`

   * If RTC present: wall-clock ISO-like format with timezone offset, e.g. `2025-10-28T20:41:03-07:00`.
   * If no RTC yet: milliseconds since boot. (User will note boot time manually.)
2. `behavior`

   * One of:

     * `VERBAL`
     * `PHYSICAL`
     * `PROPERTY`
     * `REFUSAL`
     * `SELF_HARM`
     * `REGULATED`
     * `ATTEMPT_SUPPORT` (INTERVENE tap)
     * `ATTEMPT_BOUNDARY` (INTERVENE hold)
     * `INCIDENT_END` (system-generated on close)
3. `target`

   * `"ME"`, `"SIB"`, `"OTHER"`, or `""`
   * `""` for INTERVENE rows and REGULATED rows
4. `flag`

   * `""`
   * `"threat"`  (VERBAL hold)
   * `"severe"`  (PHYSICAL hold, PROPERTY hold if implemented)
   * `"danger"`  (SELF_HARM hold)
5. `incident_file`

   * `""` if no incident is active for that event
   * Otherwise the current incident audio filename, e.g. `incident_2025-10-28T20-41-03.wav`

On incident close:

* Firmware appends a final row:

  * `behavior = INCIDENT_END`
  * `target = ""`
  * `flag = ""`
  * `incident_file = (that incident's filename)`

Requirement:

* The behavior values and flag strings in the CSV must be human-readable and stable. Do not shorten them to cryptic codes.

Requirement:

* Every event row must be flushed such that a sudden power loss will not wipe the entire log (best effort: append-then-flush on each press).

---

## 6. OLED + Haptic Feedback Behavior

### 6.1 On every front button event (including INTERVENE)

Immediately after logging:

* Trigger haptic feedback:

  * Tap → 1 short buzz (~100 ms)
  * Hold → 2 short buzzes (~100 ms each with ~100 ms gap)
* Show a 1–2 line summary on OLED for ~OLED_EVENT_DISPLAY_MS:

  * Short time (e.g. `20:41`)
  * Behavior
  * Target (in parentheses if present)
  * Flag if present (`severe`, `threat`, `danger`)
  * For INTERVENE:

    * `SUPPORT` or `BOUNDARY`
  * For REGULATED:

    * `REGULATED ❤️` (or similar gentle marker)

Example OLED confirms:

* `20:41 PHYSICAL (ME) severe`
* `20:42 VERBAL (SIB) threat`
* `20:43 SUPPORT`
* `20:44 REGULATED ❤️`

### 6.2 Between events / idle OLED display

When not showing the last event:

* OLED must display whether an incident is actively recording audio.

  * Show `REC` (or `REC ●`) if state is INCIDENT_ACTIVE or INCIDENT_COOLDOWN.
  * Show nothing / idle status if state is IDLE.
* Optional (stretch): show battery or uptime.

Requirement:

* REC indicator must be visible to the caregiver at a glance, but does not have to be flashy or externally obvious. No bright flashing LEDs intended to escalate the child.

---

## 7. Timing Constants and Robustness

Firmware must expose/obey these tunable constants:

* `LONG_PRESS_THRESHOLD_MS`
  Default: 1000 ms
  Meaning: Hold ≥ this duration is a hold event (threat / severe / danger / boundary).
  Requirement: A fast, high-force tap must NOT be mis-read as hold. Reliability here is safety-critical.
  We may need debouncing and press profile smoothing.

* `INCIDENT_COOLDOWN_MS`
  Default: 300000 ms (5 minutes)
  Meaning: If there are no new events for this long, the incident ends.
  REGULATED and INTERVENE both count as “new events” and reset this timer.

* `OLED_EVENT_DISPLAY_MS`
  Default: ~2000 ms
  Meaning: How long to show the “20:41 PHYSICAL (ME) severe” confirmation before returning to idle/REC display.

* `MAX_INCIDENT_LENGTH_MS` (stretch, but include)
  Default: 1800000 ms (30 minutes)
  Meaning: If an incident runs longer than this with constant activity, firmware can roll the audio file to a new filename to reduce corruption risk.
  After rollover, keep the same logical incident active but continue logging rows with the *new* filename. (If this is not implemented in v1, then a single file can run unrolled.)

Robustness requirements:

* Button reads must be debounced and reliable under shaky/jolty presses.
* Haptic + OLED feedback must fire on every logged event. If feedback fails, trust fails.
* Device must continue functioning after:

  * Mild drop from chest height.
  * Light lanyard yank.
  * Sweaty hands / fast presses.
* Device must not silently crash during an active incident.

---

## 8. Edge Cases and Rules

### 8.1 REGULATED outside incidents

* Pressing REGULATED in IDLE:

  * Logs `REGULATED` with `incident_file=""`
  * Does NOT start audio
  * State stays IDLE
    This captures positive repair moments even when there was no “official” escalation.

### 8.2 INTERVENE outside incidents

* Pressing INTERVENE (tap = ATTEMPT_SUPPORT, hold = ATTEMPT_BOUNDARY) in IDLE:

  * Logs the row with `incident_file=""`
  * Does NOT start audio
  * State stays IDLE
    This lets you log “I tried soothing / I tried boundary” pre-escalation without implying “this is now an incident.”

### 8.3 Transition to INCIDENT_ACTIVE from INCIDENT_COOLDOWN

* Any logged event during cooldown must:

  * Keep the same audio file
  * Reset cooldown
  * Restore state to INCIDENT_ACTIVE
    This prevents one meltdown from being split into multiple “incidents” just because there was a 60-second apology hug in the middle.

### 8.4 Clearing `pending_target`

`pending_target` must clear:

* Immediately after it’s consumed in a behavior row
* When incident ends and we go back to IDLE
* On reboot / power cycle

This prevents accidental carryover (“still thinks target=SIB 20 minutes later”).

### 8.5 Power loss mid-incident

If power dies during INCIDENT_ACTIVE or INCIDENT_COOLDOWN:

* CSV rows prior to loss are still valid.
* Audio file may be abruptly cut off.
* On reboot:

  * State must come up in IDLE.
  * No active incident should be assumed.
  * `pending_target` resets to blank.

This is acceptable for v1.3.

---

## 9. Definition of Firmware “Ready” per Spec

Firmware is considered ready for field use when:

1. State machine behaves exactly as defined:

   * IDLE ↔ INCIDENT_ACTIVE ↔ INCIDENT_COOLDOWN
   * Cooldown expiration ends the incident, logs INCIDENT_END, stops audio, clears REC
   * REGULATED and INTERVENE behave correctly in/ out of incidents

2. Button logic is correct:

   * Tap vs hold classification matches spec and is reliable under stress
   * Target tagging applies once then clears
   * INTERVENE correctly logs ATTEMPT_SUPPORT / ATTEMPT_BOUNDARY

3. CSV rows are correct and human-readable:

   * Each event row has correct `behavior`, `target`, `flag`, and `incident_file`
   * INCIDENT_END row is appended at incident close
   * I can open `events.csv` and line it up with `incident_*.wav` without guessing

4. OLED + haptics provide immediate trust feedback:

   * Every logged event gets buzz + readable OLED confirmation
   * REC indicator accurately reflects audio recording state

5. Robustness is acceptable:

   * No missed presses in real use
   * Device still works after basic drop / lanyard yank
   * Device doesn’t crash mid-incident

If any of these fail, firmware is not ready for live deployment with Stella.

