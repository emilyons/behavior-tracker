Revised Course Syllabus: Building the Stella Behavior Logger

This syllabus is designed for a self-paced learner with some coding experience but new to electronics. Each lesson includes hands-on activities and emphasizes safe, reliable engineering practices. Approximate durations are provided, and learning objectives highlight key outcomes.

Lesson 1: Introduction to the Stella Behavior Logger (Duration: ~1 hour)

Learning Objectives:

Understand the project’s purpose and features: Identify how the Stella Behavior Logger helps caregivers log behavioral incidents one-handed with minimal effort. Recognize core functions (7 behavior buttons + INTERVENE, side target buttons, audio recording, microSD storage, OLED feedback, LiPo battery power).

Connect requirements to real needs: Review key Product Requirements (e.g. one-handed logging, capturing escalation and recovery) and why the device is valuable for safety documentation.

Course overview: Outline the project milestones and lessons ahead. Emphasize the hands-on approach – the learner will build the actual device step-by-step, gaining foundational electronics and embedded programming skills along the way.

Lesson 2: Setting Up the Development Environment (Duration: ~1 hour)

Learning Objectives:

Gather hardware and tools: Collect the ESP32 dev board (with USB and LiPo charging, e.g. a Feather ESP32-S3), DS3231 RTC module, OLED display, microSD module, I2S microphone (e.g. INMP441), coin vibration motor, membrane or tactile buttons (8 front + 3 side), and supporting parts (breadboard, jumper wires, resistors, transistor/MOSFET, diode, capacitors, LiPo battery).

Install software and drivers: Set up the Arduino IDE (or PlatformIO) and add ESP32 board support. Ensure the USB driver (e.g. CH340G for the dev board) is working so the board can be programmed.

Verify programming the ESP32: Upload a simple “Blink” sketch to the ESP32’s onboard LED. Confirm the code compiles, flashes, and the LED blinks, proving the development environment and board are functioning (milestone: Blink test passed).

Microcontroller orientation: Briefly introduce the ESP32’s capabilities and why it was chosen (multiple I/O interfaces – I2C, SPI, I2S – needed for this project, built-in battery management, dual-core performance, etc.). This prepares the learner for utilizing these features in later lessons.

Lesson 3: Electronics Basics and Power Safety (Duration: ~1 hour)

Learning Objectives:

Learn electronics fundamentals: Understand voltage, current, resistance, and the concept of a circuit. Practice using a breadboard and wires to build simple circuits. For example, light an LED using a resistor and the ESP32’s 3.3V output to illustrate Ohm’s Law and why current-limiting resistors are necessary.

Establish good wiring practices: Always connect a common ground for all components and use the breadboard effectively. By the end, the learner can confidently create and debug basic circuits (e.g. LED with a pushbutton).

LiPo battery safety and power design: Understand how to handle a Li-ion polymer battery safely: never short or puncture it, and only charge it with an appropriate charger. The ESP32 board’s built-in charger will be used, with the battery plugging into the board’s JST port. Emphasize that the board’s EN (enable) pin or a dedicated slide switch should be used to power the device on/off – do not put a switch in series with the raw battery (this avoids unsafe voltage drops or unreliability). Highlight guidelines like not charging the device during an active escalation for safety.

Powering the project: If possible, demonstrate powering the ESP32 from the LiPo battery versus USB. This gives the learner a preview of portable operation and ensures they know how to charge the battery safely before later lessons.

Lesson 4: Digital Input and Output – Buttons and LEDs (Duration: ~2 hours)

Learning Objectives:

Microcontroller GPIO basics: Configure ESP32 pins as digital outputs or inputs in code. Understand the role of General Purpose I/O pins for interfacing with external components.

Build a simple input/output circuit: Wire a pushbutton to an ESP32 GPIO (one leg to the input pin, the other to ground) and use the ESP32’s internal pull-up resistor for that pin. Connect an LED (with a series resistor) to another GPIO as an output.

Write and test interactive code: Develop a sketch that reads the button state and toggles the LED when the button is pressed. Use the serial monitor to print messages on button presses for debugging and confirmation.

Understand active-low logic: Explain why using an internal pull-up means the input reads HIGH when the button is not pressed and LOW when pressed. Ensure the learner grasps this concept, as all our buttons will use active-low wiring for simplicity and stability.

Milestone: The learner can reliably detect a button press and control an LED in response. This verifies the microcontroller, wiring, and code setup are all working correctly (foundation for the device’s inputs and outputs).

Lesson 5: Debouncing Switch Inputs (Duration: ~1 hour)

Learning Objectives:

Recognize switch bounce: Learn that mechanical buttons don’t produce a clean signal when pressed – they rapidly oscillate between on/off for a few milliseconds (bounce), which can cause false multiple triggers. Understand why reliable logging demands debouncing (each press should register exactly once).

Implement a software debounce: Write a routine to filter out bounce noise. For instance, when a button state change is detected, delay a short time (e.g. 10–20 ms) or require the state to be stable for a certain interval before counting it. Introduce the idea of a debounce interval.

Test the debounce logic: Use the serial output or LED to verify that even rapid or shaky presses result in single confirmed events. For example, tap the button five times quickly and confirm you get five prints or LED toggles (and not more). If any presses register twice, refine the debounce code.

Firmware vs. hardware debouncing: Note that the hardware spec decided to handle debouncing in firmware (no extra capacitors or circuits). The learner should appreciate how software logic can solve hardware problems, an important embedded design concept.

Lesson 6: Detecting Tap vs. Hold (Long Press) Actions (Duration: ~2 hours)

Learning Objectives:

Expand input handling: Modify the button-reading code to measure how long a button is held down. Use a timestamp (from millis()) at press and release to compute press duration.

Implement tap vs hold classification: Define a threshold (1,000 ms as per specifications) to distinguish a tap (quick press) from a hold (long press). If the button is released after less than the threshold, treat it as a tap; if held longer, treat as a hold.

Simulate device behavior for holds: The Stella Logger interprets holds on certain buttons as higher-severity events (e.g. holding the PHYSICAL button logs a “severe” incident). In code, perhaps print “HOLD – severe” vs “TAP – normal” to simulate this distinction.

Test for reliability: Press and hold the test button for ~2 seconds; verify the system detects a hold event. Then do a series of very fast taps and ensure none are misinterpreted as holds. Adjust timing or debounce logic if needed so that false long-press readings are zero in quick tap scenarios. This teaches the importance of robust input timing logic.

Feedback for user: Though still early in development, design the code to give clear feedback (e.g., serial print “TAP” or “HOLD” or flash the LED differently) so the learner can observe the behavior. This foreshadows the final device’s feedback (different haptic patterns and OLED messages for holds vs taps).

Lesson 7: Introducing the Incident State Machine (Duration: ~1.5 hours)

Learning Objectives:

Understand the need for a state machine: Learn what a finite state machine is and why the device’s behavior is organized into states. Outline the Stella Logger’s three primary firmware states – IDLE, INCIDENT_ACTIVE, INCIDENT_COOLDOWN – and what each represents.

Map out state transitions: Using the Interaction & State Spec, describe the rules for transitioning between states. For example, IDLE -> INCIDENT_ACTIVE occurs when a problem behavior button is pressed (device starts a new incident and begins recording); INCIDENT_ACTIVE -> INCIDENT_COOLDOWN when no events occur for a period; and cooldown -> IDLE when the timer expires (incident ends). Emphasize special cases: pressing REGULATED or INTERVENE in IDLE should not start an incident or audio recording (by design, to avoid false incident starts).

Visualize the logic: Have the learner sketch a simple state diagram or flowchart for these states and transitions. This helps cement their understanding and will guide the firmware implementation later.

Pseudocode planning: Write pseudocode or outline in plain language how the firmware will check inputs and switch states. For instance: “If state is IDLE and a behavior button pressed -> start incident (state = ACTIVE, start recording, etc.)” and so on. Preparing this logic now will make the coding in subsequent lessons more straightforward.

Robust design thinking: Highlight how the state machine ensures the device behaves predictably even in complex scenarios (like multiple events in succession or long quiet periods). This instills good design practice by dividing functionality into manageable states.

Lesson 8: Using the OLED Display (I2C Communication) (Duration: ~2 hours)

Learning Objectives:

Interface with an I2C device: Learn the basics of the I2C bus (SDA and SCL lines with pull-ups, addresses, master/slave roles). Understand that multiple devices (OLED, RTC, etc.) can share the two I2C lines as long as they have unique addresses.

Hook up the OLED screen: Connect the 128x32 OLED to the ESP32 (SDA to ESP32’s SDA pin, SCL to SCL pin, power to 3.3V, ground to GND). Double-check wiring since I2C devices won’t work if SDA/SCL are swapped or incorrectly connected.

Display “Hello World”: Install an Arduino SSD1306 library (such as Adafruit SSD1306). Initialize the display in a sketch, taking note of its I2C address (often 0x3C or 0x3D). Write a test to print text (e.g. “Hello, World” or a simple graphic) on the OLED. Seeing custom text on the screen confirms the I2C communication and library are set up correctly.

I2C troubleshooting: If the display doesn’t turn on, use an I2C scanner sketch to detect devices and verify the address. This teaches the learner how to debug I2C issues.

Milestone: The OLED is operational and can show messages. This will later enable feedback like a “REC” recording indicator and event confirmations on the device.

Lesson 9: Integrating a Real-Time Clock (RTC) for Timestamps (Duration: ~1 hour)

Learning Objectives:

Purpose of an RTC: Understand why a real-time clock module (like the DS3231) is useful – it keeps accurate date and time, allowing log entries to have actual timestamps (wall-clock time) instead of just relative times. This aligns with stretch goals to have true time stamps without manual reference.

Connect the RTC module: Wire the DS3231 RTC to the I2C bus (SDA, SCL, 3.3V, GND) alongside the OLED. Note that I2C allows both devices on the same lines since they have different addresses. Ensure the RTC’s backup battery (if any) is in place so it can keep time when the device is off.

Use an RTC library: Install an Arduino library for the DS3231 or RTC (e.g., RTClib). Write a simple program to read the current time from the RTC and print it to serial or display it on the OLED. If the module isn’t pre-set, set the time initially in code or via a separate sketch.

Plan for timestamp logging: Once the RTC is providing correct date/time, determine how to format timestamps for the event log (e.g., ISO 8601 strings like 2025-11-16 14:30:59). Note that if no RTC were present, the design would fall back to relative timing – but with RTC, we aim to log real date/time for each event.

Optional vs. core: Acknowledge that the hardware spec treated the RTC as optional for v1, but integrating it now adds value. If a learner prefers, they can skip this and use relative timing, but doing this integration teaches how to use multiple I2C devices and handle real-time data, which is a valuable extension of the project.

Lesson 10: Working with microSD Storage (SPI Communication) (Duration: ~2 hours)

Learning Objectives:

Role of the microSD card: Understand that the SD card will store all log data (events.csv) and incident audio recordings. Emphasize the advantage of removable storage for easy data access and large capacity.

SPI bus fundamentals: Learn that microSD uses the SPI protocol (4-wire interface: MOSI, MISO, SCK, CS). The ESP32 will act as SPI master to read/write the SD card.

Wire the microSD module: Connect the SD card breakout to the ESP32 according to the pin map. For example, use a designated SPI port: SCK (clock), MOSI (data out), MISO (data in), and choose a CS (chip-select) GPIO (e.g., GPIO 13 in the hardware reference). Ensure the SD module is 3.3V logic-level; if not, use a level shifter or a module that has one built-in.

Initialize the SD in code: Include the Arduino SD library (or SDSPI) and attempt SD.begin(CS_PIN). Handle common pitfalls: the card must be FAT32 formatted, and the CS pin in software must match the hardware wiring. Provide troubleshooting tips (use Serial to print “SD init success” or error messages).

Read/write test: Write a simple file to the SD card (e.g., create test.txt and write a line of text), then read it back. This verifies that file I/O works. If successful, the ESP32 can create and handle files needed for logging.

File I/O best practices: Discuss the importance of closing files or flushing writes promptly on microcontrollers. To ensure data integrity, we will flush each log entry to the card immediately so that a sudden loss of power does not lose recent data. This is a requirement from the spec (each event must be saved to survive power loss).

Milestone: The SD card module is working – the learner can programmatically store and retrieve data on the card. This is crucial for the upcoming lessons on logging events and audio.

Lesson 11: Capturing Audio with the I2S Microphone (Duration: ~2 hours)

Learning Objectives:

Introduce I2S audio: Learn what the I2S (Inter-IC Sound) protocol is and why it’s used for digital microphones. Understand the signals: bit clock (BCLK), left-right clock (LRCLK, though our mic is mono so this acts as a frame sync), and data line. Emphasize that the ESP32 has an I2S peripheral that can interface with a digital MEMS microphone directly.

Connect the I2S microphone: Wire the I2S mic (e.g., INMP441 or similar) to the ESP32. For instance, use GPIO pins per the project pin map: one for BCLK, one for LRCLK (word select), and one for data in (must be an input-capable pin). Provide power (3.3V) and ground to the mic. Keep the wiring short and away from noisy lines, as recommended (to avoid interference).

Read audio data in code: Use the ESP32 I2S library or Arduino’s I2S functions to configure the microphone input. Set up the I2S with the desired sample rate (e.g. 16 kHz mono, 16-bit depth, per spec). Start by reading raw samples in a loop. Perhaps print out some audio sample values or compute a simple moving average to display the sound level. Verify that when you make noise near the mic, the readings change, confirming the mic is picking up sound.

Record a short WAV file: Once basic data capture works, try recording audio to the SD card. Teach the structure of a WAV file (header + PCM data). Write a routine to open a new file (e.g., test.wav), write a 44-byte WAV header, then continuously read from the mic and write the samples to the file for a few seconds. Close the file and update the header’s data length if needed.

Test audio quality: After recording, take the SD card to a computer and play the WAV file. The audio should be recognizable (speech intelligible, etc.). This validates the full pipeline: mic -> ESP32 -> SD card storage. It’s okay if quality is modest (we trade off some fidelity for smaller file size by using 16 kHz).

Milestone: The system can capture audio and save it as a file. This is a major step toward the device’s ability to record incident audio automatically.

Lesson 12: Driving the Vibration Motor (Haptic Feedback) (Duration: ~1 hour)

Learning Objectives:

Haptic feedback purpose: Understand why the device uses a small vibration motor for feedback – in a high-stress situation, a silent tactile confirmation (buzz) is felt immediately by the user without drawing the child’s attention. This design meets the requirement for discreet, eyes-free confirmation.

Motor driver circuit: Learn that the ESP32 GPIO cannot drive the motor directly because of current and voltage limits. Introduce the use of a transistor or MOSFET as a switch. If using the provided SOT23 MOSFET, explain its Gate, Drain, Source pins (with Gate to ESP32 pin via a resistor, Source to GND, Drain to the motor lead). Alternatively, a small NPN transistor (e.g. 2N2222) can be used (Base to GPIO via resistor ~1–4.7kΩ, Emitter to GND, Collector to motor).

Flyback diode protection: Understand that when the motor (a coil) turns off, it can generate a voltage spike. Use a diode (preferably the included Schottky diode for fast response) across the motor terminals to clamp this spike. Connect the diode cathode to the 3.3V side and anode to the transistor side of the motor so it shunts any back-EMF safely.

Build and test the circuit: On a breadboard, wire the coin vibration motor between 3.3V and the transistor’s output (collector or drain). Attach the diode and the resistor to the transistor’s control pin. Write a simple code to drive the GPIO HIGH for a short time to spin the motor, then LOW to stop. Test by activating the motor when a button is pressed or on a timer. The motor should buzz perceptibly.

Best practices: Emphasize keeping motor wires short and physically separated from the microphone and other sensitive circuits to reduce electrical noise. Also, note that the small motor only needs brief pulses for feedback, which saves power and reduces wear.

Milestone: The learner can control the haptic motor via the ESP32 using a proper driver circuit. This satisfies the hardware requirement for haptic confirmation on button presses, while demonstrating safe transistor use and motor integration.

Lesson 13: Building the Full Input System – All Buttons (Duration: ~2 hours)

Learning Objectives:

Scale up from one button to many: Identify all the input buttons required: six problem behavior buttons (VERBAL, PHYSICAL, PROPERTY, REFUSAL, SELF_HARM, REGULATED), one INTERVENE button, and three side target buttons (ME, SIB, OTHER). In total, 10 inputs. Plan an updated pin mapping for all these buttons, referencing the hardware spec’s guidance (avoid ESP32 pins with special boot functions, use pins with internal pull-ups).

Wire all buttons to the microcontroller: Using either the membrane switch panel or individual tact switches, connect each button’s one terminal to its assigned GPIO pin and the other terminal to the common ground rail. Enable the ESP32’s internal pull-up on each of those GPIOs in software (so the circuit remains simple, no external resistors needed for inputs). Keep the wiring organized (label each if possible) since 10 wires can be confusing – this is a good exercise in careful hardware organization.

Update firmware for multiple inputs: Extend the button-reading code to handle all 10 inputs. A simple polling loop can check each button’s state and apply debouncing. Alternatively, use interrupts for button presses if comfortable (though manage debouncing in the ISR or via timestamp). At minimum, maintain a separate debounce state for each button or reuse a debouncing function for multiple inputs.

Distinct actions for each button: Program the logic so that when each button is pressed, it prints or logs which one it was. This is a check to ensure wiring and code alignment. For instance, pressing the PHYSICAL button should result in a message “PHYSICAL pressed” and not be mistaken for another input. Verify each button in turn.

Testing all inputs: One by one, press each front button and see that the correct label is identified in serial output. Then press each side target button and ensure those register (these won’t log an event yet, just note that they were pressed). This comprehensive test ensures no cross-wiring (e.g., that pressing one button doesn’t erroneously trigger another’s code). Fix any issues now, because a solid input foundation is essential (the spec mandates no missed or incorrect presses even in fast use).

Milestone: All input buttons are wired up and recognized by the microcontroller. The device now has a “keyboard” of 10 buttons functioning. This lays the groundwork for logging actual categorized events in the firmware.

Lesson 14: Firmware – Logging Events to CSV (Duration: ~1.5 hours)

Learning Objectives:

Define event log format: Decide what information constitutes a log entry. According to the spec, each event row in events.csv should include: a timestamp, the behavior/action name, the target (if one was armed for that event), a flag for severity (if it was a hold-type event), and the incident filename if the event occurred during an incident. Design a CSV line format matching these requirements (comma-separated values).

Timestamp strategy: Now incorporate the time source. If the RTC was set up (Lesson 9), use the current date/time for the timestamp. If the RTC was skipped, use a relative timestamp (e.g., milliseconds since device boot) and note it as such. Ensure the format is human-readable (e.g., ISO date-time or seconds.milliseconds).

Implement the logging function: Write a function that takes an event’s details and appends a line to events.csv on the SD card. Use the SD library to open the file in append mode, write the formatted line, and then flush/close the file. Emphasize flushing after each write to guarantee the data is saved to the card immediately (per the requirement to survive sudden power loss during incidents).

Integrate button handling with logging: Connect the input handling (from Lesson 13) to this logging function. For now, each time a front-face button event is detected (tap or hold), log an entry with the appropriate fields. Use placeholders for any fields not yet implemented (e.g., incident filename can be blank or “N/A” if no incident concept yet, target can be blank if none armed).

Test event logging: Simulate a few events: press a behavior button and an INTERVENE, etc., then remove the SD card and open events.csv on a PC. Verify that each press created one line, the text labels are correct (e.g., “PHYSICAL, severe, SIB” etc., depending on what was pressed), and the file is properly formatted. This ensures the basic logging mechanism works before adding complexity.

Data format accuracy: Check that things like empty target fields are truly empty (not a placeholder word), that flags like “severe” only appear when appropriate, and that the CSV columns line up with spec definitions. Adjust formatting or ordering as needed.

Milestone: The device can record events to the SD card in CSV format. At this stage, pressing buttons generates logged data – fulfilling the core logging functionality of the project in a basic form.

Lesson 15: Firmware – Implementing Incident State Logic (Duration: ~2 hours)

Learning Objectives:

Combine state machine with logging: Now integrate the state machine (from Lesson 7) into the firmware so that the device’s behavior aligns with incident logic. The goal is that pressing certain buttons will transition states and manage audio recording as defined in the spec.

Start a new incident (IDLE -> ACTIVE): Code the condition where if the state is IDLE and a problem behavior button is pressed (VERBAL, PHYSICAL, PROPERTY, REFUSAL, SELF_HARM), the firmware should initiate a new incident. This means: generate a new incident ID or filename (e.g., a timestamp-based name for the audio file), start recording audio to that file (integration with the I2S routine), log the event with the incident filename attached, set state = INCIDENT_ACTIVE, and indicate recording status on the OLED (e.g., set a “REC” flag).

Maintain an active incident (ACTIVE state): Ensure that while in INCIDENT_ACTIVE, any further events (whether more behaviors, or REGULATED, or INTERVENE presses) are logged to the same incident file and keep the incident alive. Implement logic so that each new event resets a “cooldown” timer (so the incident doesn’t end while activity continues). State remains ACTIVE during continuous events.

Cooldown period (ACTIVE -> COOLDOWN): Implement the transition to INCIDENT_COOLDOWN after inactivity. If no new events occur for a short grace period after the last event, switch state to COOLDOWN and start a longer timer (e.g. 5 minutes). In COOLDOWN, the device continues recording audio and still considers the incident open, but is waiting to see if activity resumes. The OLED “REC” indicator should remain on during this time.

Incident end (COOLDOWN -> IDLE): If the cooldown timer expires with no further events, finalize the incident. Stop the audio recording and close the file, log an INCIDENT_END event to the CSV (with the incident filename), clear the incident filename from memory, and set state back to IDLE. Also update the OLED to remove the REC indicator (since we’re no longer recording).

Cooldown interruption: If a new event occurs during the cooldown period, implement the logic to cancel the cooldown and go back to INCIDENT_ACTIVE. The incident continues seamlessly (no new audio file; it keeps using the existing one). This matches the spec that any event in cooldown extends the incident.

Special case handling: Ensure that pressing INTERVENE or REGULATED while in IDLE does not start an incident or recording. In those cases, log the event normally (with no incident_file) but keep state = IDLE. Also, decide how target button presses behave in IDLE (most likely, they just arm a target and don’t affect state). Clear any armed target when an incident ends or when starting a new one, to avoid carry-over.

Testing the logic: Without worrying about actual audio for the moment, use serial prints to trace state changes. For example, simulate: IDLE -> press PHYSICAL (should print “State = ACTIVE, recording started”); press INTERVENE during active; then stop activity and simulate 5 minutes passing (perhaps shorten the timer for test) to see if it prints “State = IDLE, incident ended”. Verify that INCIDENT_END gets logged appropriately.

Milestone: The firmware now correctly manages incident state transitions and incident lifecycles. The device “knows” when an incident is in progress and when to end it based on input activity, fulfilling the core logic outlined in the state spec.

Lesson 16: Firmware – Integrating Audio Recording with Incidents (Duration: ~2 hours)

Learning Objectives:

Tie audio control to incident state: Modify the incident state code to actually start and stop the I2S microphone recording in sync with incidents. On the transition to INCIDENT_ACTIVE, begin recording audio to the new incident WAV file. This involves initializing the I2S mic (if not already running) and creating a new file on SD with a unique filename (e.g., incident_TIMESTAMP.wav). Write the WAV header and prepare to stream data.

Continuous audio logging: Ensure that while in INCIDENT_ACTIVE or INCIDENT_COOLDOWN, the mic data is being read and written to the SD card. Implement this either in the main loop with careful buffering or consider using a separate FreeRTOS task on the ESP32 to handle audio so that button responsiveness remains high. Teach the concept of double-buffering or DMA with I2S: accumulate audio samples in a buffer, then write the buffer to SD in chunks to avoid blocking too long. This introduces basic concurrency considerations on microcontrollers.

Stop recording on incident end: On the transition from COOLDOWN to IDLE (incident end), stop the I2S recording gracefully. Finish writing any buffered audio samples, close the WAV file, and if necessary update the WAV header’s length field. Ensure no file remains open to avoid corruption.

Maintain responsiveness: Test that during recording, the system still responds to button presses quickly. For example, even while writing a large audio buffer to SD, a quick button tap should be detected and logged. If any presses seem to be missed or delayed, adjust the buffer sizes or splitting of tasks (the ESP32’s dual-core capability could be leveraged to dedicate one core to audio and one to button handling). This is an advanced performance tuning exercise, highlighting the need for real-time responsiveness.

Integration test (code level): Simulate an incident end-to-end: Start an incident with a behavior press, generate a few events, then let it end. After that, inspect the SD card on a PC: there should be an events.csv with all events referencing one incident file, and that .wav file present. Play the WAV to confirm it has audio for the duration of the incident. The audio should roughly match the timeline of events (if you spoke a timestamp or label during testing, verify it).

Milestone: Audio recording is fully integrated with incident management. At this point, the device automatically records audio for each incident and stops recording when the incident ends – a key functional requirement of the Stella Logger. The core functionality (logging events with synchronized audio) is achieved.

Lesson 17: Feedback Implementation – OLED Messages and Indicators (Duration: ~1 hour)

Learning Objectives:

Provide immediate visual feedback: Implement on-screen confirmations for each logged event, per requirements (every press must give the user feedback). Decide on a concise message format for the OLED’s 128x32 display. For example, when a behavior event is logged, show the behavior name and possibly target/flag for ~2 seconds. E.g., “PHYSICAL (SIB) – severe” if a PHYSICAL hold was logged with SIB target.

Show recording status: Design a persistent indicator on the OLED for when the device is recording audio. A common approach is to show a “REC” label or a red dot icon in a corner whenever state is ACTIVE or COOLDOWN. When the device is idle (no incident recording), this indicator should be absent to avoid confusion.

Manage the OLED display state: Implement logic to handle two display modes – (1) Event confirmation mode (a recent event’s details showing for 2 seconds) and (2) Base status mode (idle or recording indicator). Perhaps use a timer or timestamp: when an event is logged, update the OLED with the event message and set a displayTimeout = now + 2000ms. After 2 seconds, revert the OLED to showing just “REC” or blank depending on state. If another event comes before 2 seconds are up, immediately overwrite the display with the new event (no need to wait out the previous timer).

Implement the code: Use the SSD1306 library to display text. You might use two lines of text: e.g., a timestamp or “REC” status on one line and the event name on another. Since the screen is small, use abbreviations or icons as needed (the PRD gave an example of including time and labels on the OLED). Ensure text is readable at a glance (e.g., use a large font for “REC” or important indicators).

Test OLED feedback: Run through a few example events: Press a behavior button and see the OLED show the correct text, then return to “REC” or blank after ~2 seconds. Check that while in an incident (recording), the “REC” label is consistently present except when an event message temporarily overrides it. End an incident and verify “REC” disappears at incident end.

Milestone: The device’s visual feedback system is complete. The caregiver will see immediate confirmation of each button press on the OLED and can always tell when the device is recording. This meets the requirement for unambiguous, real-time feedback without relying on a phone or additional device.

Lesson 18: Feedback Implementation – Haptic Alerts (Duration: ~0.5 hour)

Learning Objectives:

Add tactile confirmation: Program the vibration motor to buzz in patterns corresponding to tap vs hold events, fulfilling the haptic feedback requirement. Define patterns: e.g., a short 50 ms buzz for a tap, and two short pulses for a hold (with a brief gap in between).

Incorporate haptics into event logic: Whenever a front-face event is logged (behavior or INTERVENE), trigger the appropriate vibrate pattern. Write a helper function (e.g., vibrate(count)) that activates the motor driver GPIO accordingly: one buzz or two buzzes. Tune the duration so that it’s easily felt but not overly long (50–100 ms is usually enough for a small motor).

No haptic on target-only presses: Ensure that pressing a side target button by itself does not produce a haptic buzz. The reasoning (per design) is to keep target selection discreet – you don’t want a confirmation buzz when arming a target, as it’s not a full event and could be confusing or unnecessarily noticeable. Implement this by only calling the vibrate function when an actual event is logged to CSV (and perhaps skip if the event is just a target arm).

Test the haptic feedback: Try pressing each type of button and feel the response. For a tap of a behavior or the INTERVENE button, you should feel one quick buzz; for a hold (≥1s) of those buttons, two buzzes in succession. Verify no buzz on pressing ME/SIB/OTHER alone. This can be tested with the device in hand, and it should match the OLED feedback (i.e., whenever OLED shows an event, a buzz is felt, except for target-only presses).

Integration with user experience: Emphasize how now the device gives multimodal feedback – visually via OLED and via touch with haptics. This redundancy ensures the caregiver knows an event was logged even if they can’t look at the device (they’ll feel it) and even if they can’t feel it (they could see it). This is critical for reliability under stress.

Milestone: Haptic feedback is fully integrated. Every actionable button press yields a tactile confirmation (1 buzz or 2 buzzes), meeting the design spec for immediate haptic acknowledgement of events.

Lesson 19: Implementing Target Tagging Logic (Duration: ~1 hour)

Learning Objectives:

One-shot target mechanism: Introduce how the side target buttons (ME, SIB, OTHER) modify the next event. Pressing one of these arms a pending_target internally (e.g., store “SIB”) without immediately logging anything. The next behavior event will include that target, then the target resets. This feature lets the caregiver specify who the behavior was directed at.

Coding the target feature: In your input handling code, detect when a target button is pressed. Instead of logging an event, simply record which target was chosen (e.g., set a variable pendingTarget = "SIB"). Optionally, provide minor feedback for arming a target – maybe change a symbol on the OLED (like underline “SIB”) to show it’s armed, or a brief flash of an LED. But per spec, no haptic buzz for target presses.

Modify event logging to include targets: When logging a behavior event (VERBAL, PHYSICAL, etc.), check if pendingTarget is set. If yes, include that target string in the CSV entry’s target field, then clear pendingTarget back to none. Ensure that if no target was armed, the target field is blank. Also, if an event is REGULATED or INTERVENE, always log target as blank (those don’t get targets according to spec). In those cases, you may also clear any pending target, since an unrelated event happened.

Test one-shot behavior: Simulate sequences to verify the logic: Press SIB, then press a behavior like PHYSICAL (tap). Check that the PHYSICAL event in the CSV has “SIB” as target. Then press PHYSICAL again without a new target; confirm this second press has no target (the previous one was consumed). Repeat for ME and OTHER to ensure each works. Also test that if you arm a target but then press INTERVENE or REGULATED, the target does not apply (and ideally gets cleared).

Review rules with the learner: Emphasize that target buttons themselves do not create log entries, and they only tag the very next behavior event. This enforces that the log shows targets only in context of actual behaviors.

Milestone: Target tagging is functioning. The learner’s device can now capture who the behavior was directed at, fulfilling another key requirement of the logger. The event log will have the target column populated correctly on the appropriate rows, which adds valuable context to the data.

Lesson 20: Full System Integration Test (Bench Prototype) (Duration: ~2 hours)

Learning Objectives:

Assemble all components together: At this point, ensure all parts of the system are connected simultaneously on the breadboard or prototype board: ESP32, OLED, SD card, mic, motor, all buttons, and the LiPo battery (if not already). This might be the first time everything is running at once, so tidy up wiring and secure connections to prevent accidental disconnections during tests. Use the pin map as a checklist to verify each component’s wiring.

Follow the Test Plan systematically: Perform a thorough integration test of the device’s functionality, as if verifying a final product. This should cover:

Button presses and logging: Press each front-face button (VERBAL, PHYSICAL, PROPERTY, REFUSAL, SELF_HARM, REGULATED, INTERVENE) with a tap and observe: one haptic buzz, correct OLED message, and a new CSV entry with the correct behavior name (and no flag). Then test holds on those that support it and confirm: two buzzes, OLED indicates the flag (e.g., “threat” or “BOUNDARY”), and CSV shows the proper flag for that event. No presses should be missed during these tests, even if done quickly.

Target tagging: Press a target (e.g., SIB) then a behavior, and confirm the first event has the target in the log and on the OLED confirmation, and a subsequent behavior (without re-arming) has no target. Also ensure no log entry or buzz occurs just from pressing the target button alone.

Incident start/stop: Trigger a new incident by pressing a problem behavior from IDLE. Check that audio recording starts (OLED “REC” appears) and an incident WAV file is created. During the incident, press a mix of events (including INTERVENE and REGULATED) and verify they all log under the same incident filename. Then allow the device to sit idle until cooldown triggers incident end (you can shorten the 5 min for testing). Confirm that after the timeout: audio recording stops, an INCIDENT_END entry is added to the CSV, OLED “REC” turns off, and the state resets to IDLE.

No-incident events: While IDLE, press REGULATED or INTERVENE and ensure they log to CSV without starting any audio recording (no REC on screen).

Data integrity check: After these tests, power off and inspect the SD card on a computer. Verify that events.csv is readable, properly formatted, and entries are in the correct chronological order. Each event’s incident_file field should either be blank or match the intended incident WAV name. Play back the audio file(s) and ensure they have valid audio for their entire duration with no corruption. This confirms our file-handling (flush/close) is robust.

Basic robustness tests: If possible on the bench, test a drop scenario (with padding): with the device logging or recording, gently drop it from chest height onto a soft surface. It should continue operating without a reset. Also simulate a sudden power loss: turn off the device (or pull the USB if battery is connected) during an active incident, then turn it back on. The system should boot up in a safe state (IDLE) and the CSV should contain all events up to the cutoff (maybe missing an INCIDENT_END for that one, which is acceptable). This tests the resilience against real-world handling.

Milestone: The fully assembled prototype passes all major functional tests. The learner now has a working Stella Behavior Logger on the bench that meets the specifications for logging, audio recording, feedback, and reliability. They should feel confident that the design is sound before moving on to enclosure and final assembly.

Lesson 21: Enclosure Design and Hardware Assembly (Duration: ~2 hours)

Learning Objectives:

Design for ergonomics and safety: Review the enclosure requirements from the hardware spec – roughly a handheld size (about 12 cm × 5 cm × 2 cm) that is lightweight and can be operated with one hand. It should have a neutral appearance (no intimidating look, no flashy lights externally). Plan the layout: the six behavior buttons in a 2×3 grid on the front, with the INTERVENE button distinctly placed (often centered below the grid). The OLED should be near the top for visibility. The three target buttons will be on the side for thumb access.

Front panel layout: Sketch or mark positions for each front button on the enclosure. Ensure spacing is sufficient to avoid accidental presses of neighbors, and that INTERVENE is distinguishable by touch (perhaps larger or a different texture). If using a membrane panel with labeled buttons, align it according to the design.

Side panel layout: Position the ME, SIB, OTHER buttons vertically on the side, spaced so a thumb can find each by feel. Consider adding a tactile feature (like a small bump or shape difference) on these to differentiate them without looking.

Prepare the enclosure: Choose a method to create the case: 3D printing a custom design, repurposing a plastic project box, or laser-cutting an acrylic box. The hardware spec suggests a Hammond 1553B-size ABS case, which could be drilled and cut to fit components. Create a drilling template (on paper) to mark where to cut holes for buttons, the OLED screen, the USB port for charging/programming, and the microSD card slot. Accuracy is important so that everything lines up.

Mounting components: Plan how to secure the PCB or dev board inside the case (using standoffs or hot glue), how to fix the buttons in place (they may snap into holes or need glue/holders), and how to mount the OLED (some OLEDs have mounting holes; otherwise, a snug cutout or glue can hold it). Ensure the microphone has a clear hole or mesh opening to capture sound without obstruction. Place the mic hole where it’s not covered by a hand or likely to rub on clothing (to minimize noise).

Lanyard anchor and strain relief: As an important safety detail, attach the lanyard to the enclosure body, not the PCB. If the case doesn’t have a built-in loop, create one (for example, drill a small hole or use a screw eyelet) so that any force on the lanyard doesn’t stress the internal electronics. Similarly, secure the LiPo battery leads inside (a dab of hot glue or a zip-tie anchor) so they can’t be yanked off the board if pulled. These measures prevent internal damage during rough use (matches survivability requirements).

Assemble and re-test: Move the circuit from the breadboard into the enclosure, or onto a protoboard if making a more permanent assembly. Solder connections as needed for reliability. Once everything is placed in the case, do a full functional test before sealing it up. Sometimes wires can come loose or short during enclosure assembly, so verify all buttons, the OLED, SD, mic, and motor still work. It’s easier to fix any issue now than after closing the case.

Milestone: The device is physically assembled in its enclosure. The learner now has a handheld, battery-powered Stella Logger device. It should meet the design’s physical criteria (size, layout, durability features) and still function correctly after being put together.

Lesson 22: Final Testing and Validation (Duration: ~1.5 hours)

Learning Objectives:

Perform full device validation: Repeat the comprehensive tests from Lesson 20 on the fully assembled device to ensure nothing changed during assembly. Each front button tap and hold should register with correct feedback (OLED + haptic) and proper logging. Check that target tagging works when the device is used in-hand (press a side button then a front button – the mechanism should still function flawlessly).

Incident scenario testing: Simulate a realistic use-case: start an incident, log multiple events (with targets and interventions), then conclude it. Confirm that all events went to the same incident file and an INCIDENT_END was logged after cooldown. Verify that the OLED showed “REC” during the incident and turned off after, and that the haptic feedback was felt for each front press. This ensures the device meets the user’s needs in a typical scenario.

Data integrity and sync: Remove the SD card and inspect the data after a full scenario. The events.csv should be readable and accurately reflect the event sequence (timestamps in order, correct labels, flags, targets). Each incident WAV file should play from start to finish without error and contain the audio of the whole incident. Cross-check that if, say, a “threat” verbal event was logged, you can hear the corresponding moment in audio. This gives confidence in the synchronization of logs to audio.

Power interruption recovery: Test the device’s behavior if power is lost mid-incident. For example, turn off the device (using the EN pin switch or disconnect power) while recording. Then power it back on. It should start up in a safe state (IDLE, not recording) and not automatically resume the old incident. Check the CSV: all events up until the shutdown should be present (there might not be an INCIDENT_END for that incident, which is expected in an abrupt shutdown). This tests robustness and data safety in case of unexpected power loss – a crucial reliability aspect.

Usability under realistic conditions: Wear the device (if it has a lanyard) and try using it as a caregiver would: one-handed and mostly eyes-free. Practice pressing target + behavior by feel and verify it’s doable. Press multiple buttons in quick succession (e.g., log two different behaviors within 2 seconds) to ensure the system can keep up (no freezes, all events logged). Glance at the OLED during use to see if the text is readable within the short display window. These subjective tests ensure the device is not only functional but practical in real life.

Durability tests: If feasible, do a gentle drop test with the fully built device (onto carpet) while it’s on, then check it still works (no reboot or loose parts). Also, do a “lanyard yank” test: tug firmly on the lanyard as if a child grabbed it, and then verify the device still operates and nothing internal got damaged. These tests correspond to the survivability requirements (drop and yank) that the design must pass.

Final acceptance: Ensure all the above tests meet the criteria (e.g., zero missed button presses out of many tries, all audio files valid, etc.). If any test fails, debug and fix the issue, then repeat the test. By the end, the device should reliably fulfill all must-have requirements from the PRD.

Milestone: The assembled Stella Behavior Logger passes final validation. It’s ready for real-world trial use. The learner has successfully built a functional, safe, and robust device that meets the project goals.

Lesson 23: Troubleshooting and Support Skills (Duration: ~1 hour)

Learning Objectives:

Reflect on debugging techniques: Discuss common issues encountered and how to troubleshoot them. Reinforce the value of using serial printouts in firmware to trace execution and states (e.g., printing when state changes, or when a button is detected). This helped during development and can help in the field (one could repurpose the OLED or an LED for debugging if needed when a serial monitor isn’t available).

Hardware troubleshooting tips: Encourage the use of basic tools like a multimeter to check the hardware. For example, if a button isn’t registering, measure the voltage at the GPIO pin to ensure it goes from 3.3V to 0V when pressed (internal pull-up working). If the device isn’t powering on, check battery voltage and connections, and that the regulator isn’t in a fault (some boards have a charging/status LED to observe).

List common problems and fixes: Go over a few potential problems and how to address them: SD card errors (often fixed by re-checking wiring or formatting the card properly), OLED not displaying (verify I2C address or wiring), microphone noise (keep it away from the motor, consider adding a small filtering delay or foam over the mic), haptic causing resets (ensure the diode is in place and consider adding a capacitor to stabilize supply if needed). This helps the learner anticipate and resolve issues independently.

Subsystem isolation: Teach the strategy of isolating subsystems when debugging. If audio recording fails, create a minimal test sketch just for the mic and SD to pinpoint the issue separate from the rest of the code. If an input acts up, test it with a simple button-read program. This step-by-step isolation is crucial for effective troubleshooting in complex systems.

Maintenance and support: Ensure the learner knows how to maintain the device. For instance, if after some use a button becomes unreliable, they should be comfortable opening the case and inspecting or replacing the switch. If the battery life seems short, they might check the battery health or replace it down the line (and recalibrate any battery monitoring, if added). Emphasize that building the project also means being able to support and iterate on it – a key engineering mindset.

Test Plan as a tool: Encourage using the Test Plan not just as a final checklist but as a guide during development. It provided concrete targets and can be reused whenever changes are made to ensure nothing regresses. Adopting such validation practices in future projects will lead to more reliable outcomes.

Milestone: The learner feels confident in their ability to debug issues and maintain the device. They have a toolkit of strategies for troubleshooting hardware and firmware problems, which will serve them in this project and beyond.

Lesson 24: Project Wrap-Up and Next Steps (Duration: ~0.5 hour)

Learning Objectives:

Recap the journey: Summarize everything the learner accomplished. From basic electronics and blinking an LED, they progressed to integrating sensors, storage, and multi-threaded audio recording in an embedded system. They applied concepts of debouncing, state machines, and data logging, culminating in a complex, functional device. Recognize this as a significant achievement and reinforcement of a wide range of skills.

Revisit key design choices: Now that the device is built, reflect on why certain technologies were used, cementing the learner’s understanding. For example: Why use an ESP32? (Its ample I/O and features like I2S, plus built-in LiPo management made it ideal.) Why physical buttons instead of a touchscreen? (Tactile buttons allow eyes-off, immediate input during a crisis.) Why a digital MEMS microphone and SD card? (The I2S mic simplified audio handling with no external ADC needed, and the SD card provides large, removable storage for evidence data.) Why haptic feedback over sound? (A vibration is private and won’t escalate the situation, aligning with the “non-escalating” design goal.) Revisiting these answers helps the learner appreciate the engineering rationale behind the device.

Acknowledge safety considerations: Note how power and safety were addressed: using a LiPo battery for high energy density, with proper charging circuits; adhering to safe switching (using EN pin); strain relief on critical attachments; and ensuring no outward aggressive features (no bright LEDs or noises) that could alarm anyone. These are as important as the functional features.

Future enhancements: Encourage the learner to think of features or improvements for a Version 2. The PRD and spec hinted at some stretch goals: for instance, adding a Real-Time Clock was one (if they haven’t done it already) to get actual time stamps; displaying battery percentage on the OLED; or using the ESP32’s connectivity to maybe sync data via Bluetooth or Wi-Fi in real-time (though that introduces complexity). Mention that a natural next project could be designing a custom PCB for the logger, to shrink the size and improve durability. Also, perhaps a more polished enclosure (3D printed) or even weatherproofing it could be explored. These ideas show that there is plenty of room to iterate and build on what they’ve learned.

Real-world impact: Remind the learner of the meaningful purpose of the device. By building the Stella Behavior Logger, they haven’t just done a technical project – they’ve created a tool that can help manage and document extremely challenging situations with empathy and safety in mind. This is a device that logs not just “bad moments” but also positive interventions and recovery (regulated moments), which is a compassionate approach baked into engineering.

Encourage sharing and continued learning: Suggest the learner document their build process and results. This could be valuable for their portfolio or to help others who might attempt the project. They should feel proud of the new skills acquired and the tangible outcome they achieved – a fully functional, custom piece of assistive technology. The end of this course is really a beginning, as they can now confidently tackle new projects or modify this one with the solid foundation they’ve built.
