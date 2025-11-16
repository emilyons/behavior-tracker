

# Product Requirements Document (PRD)

**Project Name:** Stella Behavior Logger
**Version:** 1.3
**Date:** October 28, 2025
**Owner:** You

---

## 1. Purpose

This device allows a single caregiver to objectively capture safety-relevant behavior data during dysregulation in real time, with almost no cognitive overhead.

The device must let me:

* Record what happened (behavior category, severity)
* Record who it was directed at (me / sibling / other)
* Record what I tried in response (support vs boundary)
* Capture the flow of escalation and recovery with timestamps
* Capture contextual audio from the escalation, automatically

Why:

1. So I can understand patterns (triggers, escalation speed, what ends it).
2. So I can advocate for appropriate support with real evidence, not “I’m just saying this.”
3. So I don’t have to rely on memory from crisis moments.

This is not a punishment tool. It’s a safety / documentation / advocacy tool.

---

## 2. User + Environment

**Primary user:**

* A single parent or caregiver managing an intense behavioral escalation (verbal aggression, physical aggression, threats, bolting, self-harm risk), sometimes while physically preventing harm to siblings.

**Context of use:**

* Home transitions (after school, bedtime).
* Public settings (parking lot, sidewalk, store).
* High emotional intensity; yelling, physical contact, fast escalation.
* The caregiver often only has one free hand and cannot look at a phone.

**Design constraints from environment:**

* Operation must be one-handed, mostly eyes-off.
* Button press must register instantly; there’s no second attempt.
* The device must be wearable (lanyard / clipped) and light.
* It must not look like a weapon, restraint tool, or obviously “I’m recording you,” to avoid escalating the child or alarming bystanders.

---

## 3. Core Functional Requirements (Must-Haves)

### 3.1 Behavior event logging (front face buttons)

The device must have dedicated physical buttons on the front for core behavior categories. Pressing a button creates a timestamped log row immediately.

Buttons and meanings:

* **VERBAL**

  * Tap: Verbal aggression / insults.
  * Hold (≥1s): Threat / violent threat. Flag = `threat`.

* **PHYSICAL**

  * Tap: Physical aggression toward a person (hitting, kicking, scratching).
  * Hold: High-risk / injury-risk physical aggression. Flag = `severe`.

* **PROPERTY**

  * Tap: Throwing, breaking, slamming objects, using objects in dangerous ways.
  * (Optional for v1 firmware) Hold: Escalated property destruction. Flag = `severe`.

* **REFUSAL / BOLT**

  * Tap: Unsafe refusal / bolting / running off / “no, you can’t make me” in a way that creates a safety problem.

* **SELF_HARM**

  * Tap: Self-harm / self-endangerment behavior.
  * Hold: Acute/high-risk self-harm attempt. Flag = `danger`.

* **REGULATED**

  * Tap: Signs of repair/reconnection (apology, hug, “I love you,” calmer body).
    This is essential because it documents self-regulation capacity, not just “bad moments.”

Requirements:

* Each press creates a log row in `events.csv` with timestamp, behavior, optional severity flag, optional target (see Section 3.2), and the active incident file if one exists.
* Each press must generate immediate confirmation:

  * Haptic buzz (1 buzz tap, 2 buzz hold).
  * OLED text for ~2 seconds: e.g. `20:41 PHYSICAL (ME) severe`.
* Missed presses are not acceptable. Reliability here is mandatory.

---

### 3.2 Target tagging (side buttons)

The device must support capturing who the behavior was aimed at, because risk to self vs risk to sibling vs risk to caregiver matters for safety planning.

There will be three smaller buttons on the device’s side, reachable by thumb:

* ME
* SIB
* OTHER

Behavior:

1. Pressing one arms that target internally (`pending_target = ME/SIB/OTHER`).
2. The NEXT logged behavior event will include that target in its CSV row.
3. After that, `pending_target` automatically clears back to blank.

Rules:

* Target buttons themselves do not generate CSV rows.
* Targets apply only to behavior events, not to REGULATED or INTERVENE (below).

---

### 3.3 Intervention logging (INTERVENE button)

The caregiver’s actions are part of the story and must be logged.

There will be a distinct INTERVENE button on the front face.

* Tap: ATTEMPT_SUPPORT
  (Co-regulation / comfort / sensory support / snack / offering choices / validating feelings / “breathe with me.”)

* Hold (≥1s): ATTEMPT_BOUNDARY
  (Safety boundary / blocking harm / physically separating siblings / removing dangerous object / “you cannot hit her.”)

Requirements:

* These log as behavior rows with behavior values: `ATTEMPT_SUPPORT` or `ATTEMPT_BOUNDARY`, timestamped.
* `target` is always blank for these rows.
* Pressing INTERVENE during an active incident keeps that incident alive and is recorded in that incident’s audio timeline.
* Pressing INTERVENE while IDLE does NOT start a new incident or start audio. I should be able to preemptively try to calm her without acting like “this is now an escalation.”

INTERVENE is not optional or “nice later.” It is core: it documents that I actively intervene, which matters for credibility and for seeing what works.

---

### 3.4 Automatic incident creation and audio capture

The device must model an “incident” automatically without me starting/stopping anything.

Rules:

* If the device is IDLE (no active incident) and I log any *problem* behavior (VERBAL / PHYSICAL / PROPERTY / REFUSAL / SELF_HARM):

  * The device immediately:

    * Creates a new incident ID
    * Starts recording audio to a new file (e.g. `incident_2025-10-28T20-41-03.wav`)
    * Logs the event row to CSV with that filename in the `incident_file` column
    * Shows `REC` on the OLED

* While that incident is active:

  * Every logged event (including REGULATED and INTERVENE) is written to the CSV with that same `incident_file`.
  * Each new event resets a cooldown timer.
  * Audio keeps recording the whole time.

* After no new events for the cooldown period (default: 5 minutes):

  * The device stops audio, appends an `INCIDENT_END` row to CSV with that same `incident_file`, clears `REC`, and returns to IDLE.

* A new escalation later creates a new incident and a new audio file automatically.

Additional requirements:

* REGULATED during an incident counts as continued activity and keeps the incident alive. (Kids often flip between rage and apology before they’re actually settled.)
* INTERVENE during an incident (support or boundary) also keeps the incident alive.
* REGULATED or INTERVENE alone while IDLE does not start an incident or start audio.

Outcome:

* You get one audio file per real blow-up.
* You get timing of escalation, what she did, who it was directed at, what you did, and the point where it ended.

---

### 3.5 Data logging and output

The device must store:

* `events.csv`
* One `.wav` per incident (`incident_*.wav`)

Each row in `events.csv` must contain:

* `timestamp`

  * If RTC exists: wall time (e.g. `2025-10-28T20:41:03-07:00`)
  * If no RTC yet: time since boot in ms, plus you note boot time manually
* `behavior`

  * One of:

    * VERBAL
    * PHYSICAL
    * PROPERTY
    * REFUSAL
    * SELF_HARM
    * REGULATED
    * ATTEMPT_SUPPORT
    * ATTEMPT_BOUNDARY
    * INCIDENT_END (system-generated at close)
* `target`

  * "ME", "SIB", "OTHER", or "" (blank)
* `flag`

  * "threat", "severe", "danger", "" (blank)
* `incident_file`

  * "" if no active incident
  * filename of the active incident .wav otherwise

Requirements:

* I must be able to plug in via USB-C or remove SD and read both the CSV and audio on a normal laptop with no special tools.
* A reasonable human (me) must be able to interpret it that same night. If data is cryptic or spread across weird formats, it fails.

---

### 3.6 Reliability and survivability

The device must be something I can actually depend on in chaos.

Requirements:

* Every valid button press must register exactly once, with immediate haptic + OLED confirmation, even if my hands are shaky.
* Long-press vs tap must be robust: a fast hard tap must not get mis-read as “hold.”
* The device must remain usable after:

  * Being dropped from chest height onto carpet / couch / floor,
  * A light yank on the lanyard,
  * Getting handled with sweaty/shaky hands.
* The device must not silently crash and lose the log mid-incident.

If I can’t trust it in an actual meltdown, I won’t use it. Reliability is core, not stretch.

---

## 4. Physical / Industrial Requirements

### 4.1 Form factor

Requirements:

* It must be reasonably comfortable to hold in one hand and/or wear on a lanyard.
* Target envelope for v1:

  * Height: ~110–130 mm (4.3–5.1")
  * Width: ~45–55 mm (~1.8–2.2")
  * Thickness: ~18–22 mm (~0.7–0.9")
  * Mass: ideally ≤ ~80 g

This is “short TV remote / slim personal device,” not “full phone,” not “brick.”

### 4.2 Button layout and ergonomics

Requirements:

* Front face must include:

  * Six main behavior buttons arranged in a stable 2×3 grid:

    * Row 1: VERBAL | PHYSICAL
    * Row 2: PROPERTY | REFUSAL
    * Row 3: SELF_HARM | REGULATED
  * A distinct INTERVENE button below or offset, visibly and tactilely unique (different size/color/texture).
    INTERVENE is your action, not her behavior — it should *feel* different.

* Side edge must include:

  * Three small vertically stacked target selectors: ME / SIB / OTHER, reachable by thumb.
  * Spacing / texture must allow eyes-off selection (you can feel them apart).

* Buttons should be low-profile (membrane/tact style), not giant arcade domes, to keep thickness low and avoid snagging on clothing or escalating attention.

### 4.3 Screen and indication

Requirements:

* A small OLED (~0.96", 128x64) must be mounted near the top/front.
* After any event press, OLED must briefly display the last event and target/flag.
* When idle between events, OLED must clearly show whether audio is currently recording (`REC`) so I always know.

### 4.4 Lanyard and mass distribution

Requirements:

* The lanyard anchor must be connected to the enclosure shell, not directly to PCB or battery leads.
* Internal mass (battery, ESP32 board) must be reasonably centered so the device hangs flat and doesn’t constantly flip/twist.
* The mic port must not be blocked by the lanyard.

### 4.5 Visual profile / escalation safety

Requirements:

* The device should look like a neutral personal tool:

  * It should not look like a weapon, restraint, or punishment tracker.
  * It should not broadcast “I am actively recording you” in a way that escalates the child in the moment.
* The REC indicator is for the caregiver’s awareness, not a big flashing light for the child.

---

## 5. Stretch Features (Nice-to-have, not required for v1)

These are allowed to slip if they interfere with the must-haves:

* Real-time clock (RTC) for true wall-clock timestamps without manual boot-time notes.
* Battery % readout on OLED.
* Better-sealed / wipeable faceplate for durability, sweat/tears/spill resistance.
* Tiny haptic “tick” when a target (ME / SIB / OTHER) is armed.
* Automatic long-incident rollover (e.g. split audio if one incident lasts >30 minutes, so files don’t corrupt).
* Outer enclosure polish (rounded edges, calmer colors, non-medical feel).

---

## 6. Non-Goals (Explicitly out of scope for v1)

We are not doing:

* WiFi / Bluetooth syncing
* Phone app or cloud account
* Background always-on audio capture
* AI categorization, threat scoring, or live behavior coaching prompts
* Touchscreen UI
* Requiring the caregiver to type narrative notes during escalation

Those either increase complexity, create privacy problems, or make the device harder to use in crisis.

---

## 7. Safety and Ethics Requirements

Requirements:

* Audio only records during an active incident triggered by a logged problem behavior. No passive all-day surveillance.
* REC state must be clearly visible on the OLED whenever incident audio is rolling.
* Data (CSV + audio) must be exportable and storable privately. This is sensitive safety/health documentation. I choose who sees it.
* The device is a support/advocacy tool. It should not frame the child as “the problem.” It should reflect escalation *and* repair, and it should record caregiver intervention attempts.

---

## 8. Definition of Done for v1.3 Prototype

The device is considered ready for first real-world use when all of these are true:

1. **Logging works:**

   * Tapping a behavior button logs that event, creates haptic feedback, and shows confirmation on the OLED.
   * Holding where defined (VERBAL, PHYSICAL, SELF_HARM, etc.) logs the correct severity/threat/danger flag.
   * REGULATED logs calming/repair moments.
   * INTERVENE logs ATTEMPT_SUPPORT (tap) and ATTEMPT_BOUNDARY (hold).

2. **Target tagging works:**

   * Pressing ME / SIB / OTHER applies that target to exactly the next event row, then auto-clears.
   * Target is optional — I can still log without it.

3. **Incident logic works:**

   * First problem behavior from IDLE starts a new incident, starts audio, and shows REC.
   * All following events (including REGULATED and INTERVENE) attach to the same incident file and keep it alive.
   * After 5 minutes of no new events, the device stops audio and logs INCIDENT_END.
   * New escalation later creates a new incident and new audio file.

4. **Data is usable by me that same night:**

   * I can get `events.csv` + `incident_*.wav` off the device (SD or USB).
   * I can open `events.csv` in a normal spreadsheet and play the matching .wav.
   * I can reconstruct “what happened, to whom, how bad it got, what I tried, and how it ended” without guessing.

5. **Physical usability and reliability are acceptable:**

   * I can hold or wear it without fatigue.
   * I can operate it one-handed in motion, mostly eyes-off.
   * It survives being dropped or yanked and still functions.
   * It does not obviously escalate the child just by existing.

If any of the above are not true, v1.3 is not ready for live field use.
