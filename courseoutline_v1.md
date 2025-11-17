Course Outline: Building the Stella Behavior Logger
Lesson 1: Introduction to the Stella Behavior Logger Project

Duration: ~1 hour

Overview of the Stella Behavior Logger’s purpose and features. Discuss how it helps caregivers record safety-relevant behavioral incidents in real time with minimal effort.

Review the device’s core functions: 7 front buttons for behaviors (plus a distinct INTERVENE button), 3 side buttons for targets, automatic audio recording of incidents, microSD data storage, OLED screen for feedback, and LiPo battery power.

Examine key requirements from the Product Requirements Document (PRD) and how the project meets a real user need for crisis documentation and advocacy (e.g. one-handed logging, capturing escalation and recovery).

Outline the course structure and project milestones. Emphasize that this is a hands-on, project-based journey — the learner will build the actual device step by step, while learning foundational electronics and embedded programming concepts along the way.

Lesson 2: Setting Up Your Development Environment

Duration: ~1 hour

Gather all required hardware: an ESP32 development board (with USB and LiPo charging, e.g. an Adafruit Feather ESP32-S3), the Elegoo Mega Starter Kit components (breadboard, jumper wires, resistors, LEDs, pushbuttons, etc.), and additional parts for the logger (microSD card module, 0.96" OLED display, I2S microphone breakout, coin vibration motor, LiPo battery).

Install the Arduino IDE (or an alternative like PlatformIO) and add ESP32 board support. Verify you can program the microcontroller by uploading a simple “Blink” test to the ESP32’s onboard LED.

Introduction to the microcontroller (ESP32): its capabilities and why it was chosen. Highlight that it’s a powerful MCU with ample I/O and peripherals (I2C, SPI, I2S) to support the OLED, SD card, and digital mic, and it often includes onboard battery management for LiPo power.

Milestone: Development environment validated – you can compile and flash a basic program to the ESP32 (the test LED blinks as expected).

Lesson 3: Electronics Basics and Safety

Duration: ~1 hour

Cover fundamental electronics concepts (voltage, current, resistance) needed for the build. Use the Elegoo kit’s resistors, LED, and a button to illustrate Ohm’s Law and why we need current-limiting resistors for LEDs.

Demonstrate proper use of a breadboard and wiring techniques. Emphasize a common ground for all components and show how to create simple circuits on the breadboard (e.g. powering an LED from the ESP32’s 3.3 V through a resistor).

Discuss LiPo battery safety in detail: characteristics of LiPo batteries and precautions for use. Emphasize not to short or puncture the battery, and to charge it only with an appropriate charger. Warn against charging or using the device in a hazardous scenario (e.g. don’t charge it while it’s being actively used during an escalation).

Explain the device’s power design. The ESP32 dev board has an onboard charger/regulator for the LiPo, so the battery will be connected directly to the board’s battery port. Highlight the recommendation to use the board’s EN (enable) pin or a separate slide switch to turn the device on/off – not to put a switch in series with the raw LiPo (which can be unsafe or unreliable).

Lesson 4: Digital Input and Output – Buttons and LEDs

Duration: ~2 hours

Introduce microcontroller GPIO (General Purpose Input/Output) pins and how to configure them as inputs or outputs in Arduino code.

Using the kit’s components, build a simple circuit: connect a pushbutton to an ESP32 GPIO (one side of the button to the GPIO pin, the other side to ground). Enable the ESP32’s internal pull-up on that input pin in software for a stable default state. Use an LED (with a resistor from the kit) on another pin as an output.

Write a program that reads the button state and toggles the LED when the button is pressed. Observe the behavior using the serial monitor (print messages on button presses) to see the input changes in real time.

Explain active-low input logic: because we use an internal pull-up, the input reads HIGH when idle and goes LOW on a button press (grounding the pin). Ensure the learner understands why the pull-up resistor and wiring to ground make the circuit simple and reliable.

Milestone: You can reliably detect a button press and control an LED in response. This basic input/output test confirms the microcontroller, IDE, and wiring setup are all working.

Lesson 5: Debouncing Switch Inputs

Duration: ~1 hour

Discuss the problem of switch bounce – the rapid, noisy on/off signals a mechanical button produces when pressed. Emphasize that reliable logging is critical: the device cannot afford to miss presses or register false multiple presses. In other words, every button press should count exactly once.

Implement a software debounce solution for the button. One simple approach: when a button state change is detected, delay for a short period (e.g. 10–20 ms) or check again after a few milliseconds to confirm the state. Alternatively, use a state-machine approach (ignore changes that happen within 20 ms of a prior change). Introduce the concept of a debounce interval.

Test the debouncing by rapidly pressing the button (or even “vibrating” it with your finger). Use serial printouts or LED toggles to verify that a single quick press only registers once. Apply the test plan’s fast tap scenario: e.g. press a button five times in quick succession and ensure you get five distinct events and 0 misread as a long-press.

Note that per the hardware spec, we are not adding extra capacitors or hardware for debouncing – it’s handled in firmware. This exercise underscores the importance of robust software design to ensure clean input signals.

Lesson 6: Handling Tap vs. Hold (Long Press) Actions

Duration: ~2 hours

Introduce the concept of interpreting both tap and long-press actions on the same button. In the Stella Logger, a >1 second hold on certain behavior buttons is treated as a more severe event (for example, holding the PHYSICAL button logs a high-severity incident).

Extend the button-reading code to measure press duration. Use a timestamp from millis() when the button is pressed, and when it is released, calculate the elapsed time. If the press lasted longer than the threshold (1,000 ms according to the spec), interpret it as a “hold” event; otherwise as a normal tap.

Verify the logic with tests: press and hold the button for ~2 seconds and check that the system recognizes a long-press. Then do quick taps and ensure they are never misclassified as holds. The test plan calls for zero false holds under rapid tapping – adjust your debounce or timing logic if needed to achieve this.

Provide feedback in the test code (e.g., print “TAP” or “HOLD” to serial, or blink the LED in different patterns) so the learner can confirm the detection is working reliably. This will mirror the final device behavior where different feedback (haptic and on-screen flags) occurs for holds vs taps.

Lesson 7: Introducing the State Machine for Incident Logging

Duration: ~1.5 hours

Present the idea of a finite state machine to manage the device’s overall behavior. The Stella Logger has three primary firmware states: IDLE, INCIDENT_ACTIVE, and INCIDENT_COOLDOWN. Explain what each state represents in context:

IDLE – no active incident, waiting for a problematic behavior to be logged (audio not recording).

INCIDENT_ACTIVE – an incident is in progress (audio recording ongoing, events are part of this incident).

INCIDENT_COOLDOWN – a temporary state after a flurry of activity; audio still recording, but will stop if no new events occur within the cooldown period.

Using the Interaction & State Spec, walk through the rules for state transitions:

IDLE -> INCIDENT_ACTIVE: occurs when a problem behavior button (one of the main five) is pressed while idle. This should immediately start audio recording and mark the start of a new incident.

INCIDENT_ACTIVE -> INCIDENT_COOLDOWN: occurs when things go quiet – i.e., after an event, no further events occur for a short initial period, trigger the cooldown timer.

INCIDENT_COOLDOWN -> INCIDENT_ACTIVE: if any new event happens during cooldown, go back to active (the incident is still ongoing).

INCIDENT_COOLDOWN -> IDLE: if the cooldown timer expires with no new events, end the incident (stop recording, log an INCIDENT_END).

Emphasize special cases from the spec: pressing REGULATED or INTERVENE while IDLE should not start a new incident or audio recording – the device only starts recording on actual problem behaviors. This is by design to avoid false incident starts.

Have the learner sketch a simple state diagram or flowchart for these transitions. Writing pseudocode for the state machine (using a state variable and a switch statement or if/else ladder) will prepare them to implement this logic in firmware later.

Lesson 8: Using the OLED Display (I2C Communication)

Duration: ~2 hours

Introduce the 0.96" OLED screen and the I2C protocol it uses. Explain that I2C (Inter-Integrated Circuit) is a two-wire communication protocol (SDA for data, SCL for clock) that allows multiple devices to connect to the same two pins, each with a unique address.

Connect the OLED to the ESP32 using the kit’s jumper wires: tie the OLED’s SDA line to the ESP32’s SDA pin (for many ESP32 boards this is GPIO 21) and SCL line to the ESP32’s SCL pin (e.g. GPIO 22). Also connect the OLED’s VCC to 3.3 V and GND to ground. Double-check these connections, as wiring errors on I2C can silently fail.

Install an Arduino library for the OLED (for example, Adafruit SSD1306 if the display uses an SSD1306 driver). In a test sketch, initialize the display and try drawing text or graphics. For instance, display a “Hello, World” message or a simple bitmap.

Cover the concept of I2C addresses: use an I2C scanner or documentation to find the display’s address (commonly 0x3C or 0x3D for these small OLEDs). Ensure the code is using the correct address to initialize the display.

Milestone: The OLED lights up and displays custom text/graphics. At this stage, the learner can make the screen show messages, which will be important for giving feedback (like a “REC” indicator or event confirmations) in the final device.

Lesson 9: Working with microSD Storage (SPI Communication)

Duration: ~2 hours

Explain the role of the microSD card: it will store the event log (events.csv) and incident audio recordings. Introduce SPI (Serial Peripheral Interface) as the protocol used by SD cards. SPI uses separate lines for data in, data out, a clock, and a chip select. The ESP32 will act as the master and the SD card as a slave device.

Connect the microSD card module to the ESP32. Follow the hardware spec’s guidance for pin connections: e.g., connect the SD module’s CS (chip select) to a chosen GPIO (like GPIO 13), SCK (clock) to the ESP32’s SPI CLK, and MOSI/MISO to the ESP32’s SPI data lines. (Many ESP32 dev boards label a default SPI bus on their pins documentation.) Ensure the SD module (perhaps from the Elegoo kit) is 3.3 V logic compatible or use a level shifter if not.

In Arduino, include the SD library and attempt to initialize the SD card. Handle common pitfalls: the card must be formatted as FAT32, and the CS pin in SD.begin(csPin) must match your wiring. Provide troubleshooting tips, like using the serial monitor to print a success/failure message after initialization.

Write a simple test: create a file on the SD (e.g., test.txt), write a line of text, close it, then reopen it to read the contents back. This tests both writing and reading functionality. If successful, the environment for data logging is set.

Discuss file I/O best practices on microcontrollers: always close files (or at least flush writes) promptly after writing data to ensure the data is actually saved to the card. Emphasize that a sudden power loss could otherwise corrupt buffered writes. (This foreshadows the need to flush each event write to survive abrupt shutdowns.)

Milestone: The ESP32 can create, write, and read files on the microSD card. This capability is crucial for logging events and storing audio in the upcoming lessons.

Lesson 10: Capturing Audio with the I2S Microphone

Duration: ~2 hours (may span multiple sessions)

Introduce the I2S digital microphone (e.g., SPH0645 or ICS-43434 breakout). Explain the basics of I2S (Inter-IC Sound): it’s a protocol specifically for streaming audio data. It uses a bit clock (BCLK) to pace the audio bits, a word select (LRCLK) to denote left/right channel or frame, and a data line for the audio bits. The ESP32 has an I2S peripheral that can interface with these mics directly.

Connect the I2S mic to the ESP32. Based on your ESP32 board, choose appropriate pins for I2S: one for BCLK, one for LRCLK (also called WS), and one for Data In. For example, if using the pin map suggestion, you might use GPIO 12 for BCLK, GPIO 15 for LRCLK, and GPIO 36 (an input-only pin) for Data. Wire 3.3 V and GND to the mic as well. Keep the wiring short if possible, as long microphone wires can pick up noise.

In Arduino, use the I2S library or an audio library to configure the I2S peripheral. Set it to receiver mode (to get data from the mic) with the desired sample rate and bit depth – e.g., 16 kHz, 16-bit mono, as the spec suggests. Explain that 16 kHz is chosen to balance audio quality with storage size, and even 8 kHz could be acceptable if performance is an issue.

Write a test that reads samples from the mic. This could be as simple as reading raw samples in a loop and printing out the amplitude (to see numbers) or computing a moving average to display sound level. If the values change when you make noise or speak near the mic, it’s working.

Next, attempt to record a short WAV file to the SD card. Explain the WAV file format basics (header + raw PCM data). Have the code open a file (e.g., test.wav), write a WAV header (44 bytes), then stream a few seconds of audio data from the mic into the file. After recording, update the header with the proper data length (or use a library that handles WAV format for you).

Milestone: You can record a brief audio clip and play it on a computer. Verify that the file plays and the sound is recognizable. The quality will be limited by the sample rate and mic, but speech should be intelligible and the file should not be corrupted. This proves the critical audio path (mic -> ESP32 -> SD) works.

Lesson 11: Driving the Vibration Motor (Haptic Feedback)

Duration: ~1 hour

Introduce the concept of haptic feedback and why our device uses a vibration motor for confirmation. In high-stress situations, a tactile confirmation (a buzz) can be felt without looking, and it’s discreet (doesn’t further agitate the child as a loud beep might).

Discuss the need for a transistor driver and flyback diode: the ESP32’s GPIO cannot source enough current to drive even a small motor directly, and the motor’s coil can generate voltage spikes when turned off. We use a small NPN transistor (like a 2N2222 from the kit) as a switch, and a diode across the motor to protect against voltage spikes.

Build the motor driver circuit: connect the coin vibration motor (or a small DC motor from the kit as a proxy) between the 3.3 V supply and the transistor’s collector. The transistor’s emitter goes to ground. Solder or connect a diode across the motor terminals (cathode to 3.3 V, anode to the transistor collector) to clamp voltage spikes. Use a resistor (e.g. 1 kΩ – 4.7 kΩ) on the transistor’s base, and connect that base to an ESP32 GPIO.

Write a simple test to activate the motor. For example, make it buzz briefly when a button is pressed or on a timer. Ensure the transistor is switching properly by feeling or seeing the motor spin. The vibration motor should provide a gentle buzz. If using a regular DC motor for testing, you’ll see it spin – that confirms the circuit.

Emphasize good practices: keep the motor’s wires short and if possible, route them away from the microphone and other sensitive lines, as the motor can introduce electrical noise. In the final assembly, mounting the motor firmly will help the buzz be felt but also dampen any sound it makes.

Lesson 12: Building the Input Subsystem – All Buttons

Duration: ~2 hours

Now expand from one button to the full array of input buttons the device requires. Identify all needed inputs: six behavior buttons (VERBAL, PHYSICAL, PROPERTY, REFUSAL, SELF_HARM, REGULATED), the INTERVENE button, and three side target buttons (ME, SIB, OTHER). That’s 10 buttons in total.

Plan a pin mapping for these buttons on your ESP32 board. Consider the hardware spec’s advice: avoid using any pins that have special boot functions (on classic ESP32, GPIO 0, 2, 15, etc., should be avoided for inputs held low at boot). Also ensure the chosen pins don’t conflict with your I2C, SPI, or I2S connections. Use pins that have internal pull-ups available and are not input-only (except for the ones purposely used for mic or SD MISO). Document this pin map for reference.

Wire up the buttons on a breadboard or proto-board. You can use the kit’s momentary pushbuttons (and any extra you have) to represent each input. One side of each button should go to a common ground rail, and the other side to its assigned GPIO pin (as you did with the single button). Enable the internal pull-up for each in software, making them all active-low inputs. Verify connections carefully – ten wires can get messy, so labeling or a diagram helps.

Update your firmware to handle multiple buttons. You can structure the code to check each button in the main loop (polling) or attach interrupts to each pin for detection. A simple approach is to poll in a fast loop and debounce each button’s state change (perhaps reuse or modularize your debounce logic from earlier). Ensure you differentiate each button (e.g., by index or by separate handler code per pin).

Test each button input one by one. For each button, press it and observe output (print which button was pressed to serial). Make sure each button is correctly identified and there are no cross-wiring issues (e.g., pressing one inadvertently triggers another in your code). This step is critical to verify the wiring and pin assignments before integrating the full logic.

Milestone: All input buttons are wired and recognized by the microcontroller. You now have a working “keyboard” of 10 physical inputs. The foundation is laid to log different events depending on which button (and tap/hold) is pressed.

Lesson 13: Firmware – Logging Events to CSV

Duration: ~1.5 hours

Implement the event logging system for button presses. First, define what data an “event” should include. According to the spec, each event log row needs: a timestamp, the behavior (or action) name, the target (if one was armed), any flag (for holds like threat/severe), and the incident file name if applicable. We’ll use a CSV text format for the log.

Decide on the timestamp strategy: since we don’t have an RTC in v1, use the time since device boot (in milliseconds) or a running seconds count as the timestamp. (Optionally, note that in the future adding an RTC could give real date/time stamps.) Include a way to indicate in the log if the times are relative.

Whenever a button event occurs (a tap or hold determined in your input handling code), create the corresponding log entry. For example: "12345,PHYSICAL,ME,severe,incident_2025-11-16T13-00-00.wav" could be a row meaning at 12.345 s after boot, a PHYSICAL event directed at SIB (target “ME” in that row meaning caregiver themselves? Double-check target logic), flagged severe, during the given incident file. Follow the exact format rules from the spec for each field (e.g., empty target as blank, not a placeholder word).

Append the line to events.csv on the SD card. Ensure you add a newline character and flush the file buffer or close the file after each entry. Flushing on every write is a bit less efficient but much safer in case of power loss. (The hardware guide explicitly says to flush per row to survive sudden power cuts during incidents.) This aligns with the test plan expectation that if the device loses power mid-incident, all events up to that moment are still in the CSV.

Test the logging functionality by simulating a few events. You could call the logging function manually in code or temporarily map a couple of your buttons to directly log an event for testing. Then eject the SD card and open events.csv on a PC. Verify that each row is correctly formatted, comma-separated, and human-readable (no cryptic codes – the log should use the actual labels like “PHYSICAL” or “ATTEMPT_BOUNDARY”). This test ensures that your file writing works and the format is correct.

(If an incident is active, include the incident filename in the log entries; if not, the incident_file field is blank. For now, you can use a placeholder or empty string since we haven’t implemented incidents yet. We will integrate that soon.)

Introduce the idea of an INCIDENT_END log entry. Plan to log a special row when an incident ends, with behavior INCIDENT_END and the incident file name, and no target/flag. We’ll generate this in the incident-handling code next, but it’s good to be aware of it while structuring the CSV.

Lesson 14: Firmware – Implementing Incident State Logic

Duration: ~2 hours (may span multiple sessions)

Now merge the state machine (from Lesson 7) with the input and logging system. This is where the device’s brains come together. Implement the following logic in your firmware:

Start a new incident: If the current state is IDLE and a problem behavior button is pressed (VERBAL, PHYSICAL, PROPERTY, REFUSAL, or SELF_HARM), then initiate a new incident. This means: generate a new unique incident ID or filename (e.g., create incident_<timestamp>.wav), start audio recording to this file, log the event with the incident_file field set to that filename, and switch state = INCIDENT_ACTIVE. Also, set a flag or variable so the OLED knows to show the “REC” indicator.

Continue an incident: If in INCIDENT_ACTIVE and another event happens (could be any front button, including another behavior, REGULATED, or INTERVENE), simply log it to the same incident file (don’t create a new file). Every event in active or cooldown should reset the cooldown timer clock. State stays INCIDENT_ACTIVE on button presses because the incident is ongoing.

Enter cooldown: In your main loop, if state is INCIDENT_ACTIVE and a certain amount of time passes with no new events (maybe a few seconds of quiet after the last event), switch state to INCIDENT_COOLDOWN to begin the final countdown. (The exact spec says it can transition immediately or after a brief idle; what matters is that the cooldown timer starts.) Start a timer for the cooldown (spec default is 5 minutes). The device should continue recording audio during cooldown, and “REC” stays on the screen.

Cooldown expiration: If state is INCIDENT_COOLDOWN and the 5-minute timer expires with no new events, then end the incident. This entails: stop the audio recording and close the file, write an INCIDENT_END row to the CSV with the same incident filename, clear the incident_file variable (no incident active), and set state back to IDLE. Also, remove the “REC” indicator from the OLED since we’re idle.

Cooldown interrupted: If state is INCIDENT_COOLDOWN and a new event occurs (meaning the situation escalated again), cancel the cooldown and go back to INCIDENT_ACTIVE. The incident continues as if it never was about to end – keep the same audio file open and logging to it. Reset the cooldown timer for the next lull.

Code these behaviors carefully. It might help to structure your loop such that it always checks for state transitions after handling any button events. For timing, you can use millis() to track last-event time and compare to a threshold (300 000 ms for 5 minutes) to decide when to end an incident.

Handle special cases: If the user presses INTERVENE or REGULATED while in IDLE, according to spec, you do log the event but you do not start recording or change state. Make sure your logic respects this: these events should be treated as standalone logs in IDLE state. Similarly, pressing a target button in IDLE does nothing by itself (it just arms the target variable, which we’ll handle soon).

Test the state logic without worrying about audio initially (you can simulate the audio actions with prints). For example, simulate: press PHYSICAL (state should go to ACTIVE), then press INTERVENE (state stays ACTIVE), then wait >5 min (you might shorten the timer for testing) and see if state goes back to IDLE and an INCIDENT_END is recorded. Use serial output generously to trace state changes and timers during testing.

Milestone: The firmware can correctly start and end an incident based on input timing. In essence, the device now knows when an incident is happening and manages the incident lifecycle as per the specification.

Lesson 15: Firmware – Integrating Audio Recording with Incident State

Duration: ~2 hours (may require multiple iterations)

Now that the state transitions are working, integrate the actual audio recording control into those transitions:

On IDLE -> INCIDENT_ACTIVE: start recording audio to the new incident file. In practice, initiate the I2S mic and open a new file on the SD card with the incident filename. Perhaps spin up a task to continuously read from I2S and write to the file (the ESP32 can handle this with a separate FreeRTOS task, or you can do it in smaller chunks in the loop). If using Arduino, ensure the SD card is writing in a streaming manner. Write the WAV header at the start of the file.

On each event during the incident, you’re already writing to CSV; just ensure the audio task keeps running concurrently. The microcontroller needs to multitask: keep reading mic data and handling button events. Because writing to SD can be slow, use buffering: collect audio samples in a buffer (a few kilobytes) and write that buffer to SD when full, so that button handling is not stalled too long. A double-buffer scheme or using the I2S built-in DMA can help. This part is advanced; you can simplify by lowering sample rate or using shorter recordings for testing.

On INCIDENT_COOLDOWN -> IDLE (incident end): stop the audio recording. Finish writing any buffered data, then close the WAV file. If needed, go back and patch the WAV header with the final file size (some libraries handle this automatically when closing). This ensures the audio file is not left corrupt or open if the incident ends.

Consider performance issues: if writing audio in real-time, make sure the loop still checks for button presses frequently. The ESP32 is quite capable, but bugs like writing a large buffer in one chunk could block for tens of milliseconds. You may test that rapid button presses (like the multi-press test) still all get logged while recording audio. Optimize by using smaller write chunks or even the ESP32’s dual-core (one core handles audio, the other core the main loop – this can be achieved with the Arduino Task API or using an interrupt-driven approach). This is an opportunity to introduce basic real-time programming considerations.

Test the fully integrated system: perform a simulated incident from start to end:

Press a problem behavior (incident starts, “REC” on, audio file created).

During the incident, press a few more buttons (including maybe a target + behavior sequence, an INTERVENE, etc.).

Wait for the cooldown to expire and incident to end (or manually trigger an end by simulating time if testing).

After it ends, remove the SD card and inspect the files on a computer. You should see an events.csv with all the events (all sharing the same incident_file name), and a .wav file for that incident. Play the WAV file on the PC – verify it plays through without issue. You should ideally hear all the events you logged (if you spoke or made noise to mark them). For example, you might say “event one” aloud when pressing the first button, and you can correlate that in the audio and the CSV timestamp.

Milestone: The core functionality is achieved – the device logs events to the CSV and records synchronized audio for the duration of an incident. This transforms the project from a simple logger to a full behavior recording device. (There may be some fine-tuning needed for performance, but conceptually everything is in place.)

Lesson 16: Feedback Implementation – OLED Messages and Indicators

Duration: ~1 hour

Now implement the user feedback on the OLED screen. According to the requirements, the device should give immediate feedback for each event logged and also indicate recording status clearly. Plan the OLED display modes:

Event confirmation: When an event is logged, display a brief message (around 2 seconds) showing what was logged. For example, if the user tapped the PHYSICAL button and had SIB armed as target, the OLED might show: PHYSICAL (SIB) (and maybe the flag if it was a hold, e.g. add severe). The PRD suggests a format like time and the event/flag, but since the caregiver likely isn’t reading the exact time in the moment, you can focus on the event text. Use clear abbreviations if needed to fit the 0.96" screen (which is typically 4 lines of text).

Recording indicator: When not showing a recent event, the OLED should show the recording status. Typically, display a “REC” icon or text in one corner when the incident is active or in cooldown. When the device is idle (no incident), you might show nothing or a standby message. This way the user can glance and see if it’s currently recording.

Implement a small routine or state for the OLED: e.g., have a boolean that indicates “showing last event message.” When a new event happens, update the display with the event details and set a timer (2000 ms per spec). After the timer, revert the display to the base state (which is “REC” if in an incident, or blank/idle screen if not).

Be mindful of the 2-second confirmation window – ensure that if another event comes quickly, you update the display again immediately (possibly overriding the older message). The haptic feedback plus OLED should together confirm every button press.

Test the OLED feedback by generating events and watching the screen. For a hold, ensure the special flag is shown (e.g., after holding PHYSICAL, maybe display “PHYSICAL (severe)”). For a target-tagged event, include the target in the message (the format Behavior (TARGET) can work). The test plan expects the user can catch these in a quick glance, so use sufficiently large text or clear labeling.

Verify the recording indicator as well: start an incident and ensure “REC” stays visible (perhaps blinking or with an icon) whenever the device is in ACTIVE or COOLDOWN state, except when temporarily overridden by a 2-second event message. When the incident ends, confirm that the “REC” indicator disappears (so the user isn’t misled into thinking it’s still recording).

Milestone: Visual feedback is fully integrated. The user of the device will get immediate confirmation on the screen for every log (preventing ambiguity about whether a press registered) and a continuous indication that audio is being recorded during an incident.

Lesson 17: Feedback Implementation – Haptic Alerts

Duration: ~0.5 hour

Integrate the vibration motor feedback with the event logic. Define the haptic feedback patterns as per requirements: a single short buzz for a tap, and two short buzzes for a hold. This was mentioned in the PRD and test plan and is a crucial part of confirmation.

Implement these patterns in code. You might create a helper function like vibrate(pattern) where pattern could be 1 or 2, and it toggles the motor driver pin accordingly: e.g., for one buzz, drive the pin HIGH for, say, 50 ms then LOW; for two buzzes, do 50 ms HIGH, 50 ms LOW, 50 ms HIGH, then LOW. The exact timing can be tuned so it’s noticeable but not too long.

Ensure that target button presses do not trigger haptic feedback. By design, pressing a side target should arm the target silently (to keep the device discreet; a buzz might confuse the situation when you’re just selecting a target). So only the front face buttons (behavior and intervene) cause a buzz. Implement this check in your input handling: perhaps call the vibrate function only when an event is actually logged to CSV (and if that event is not a pure target press).

Test the haptic feedback by pressing various buttons: for each tap of a behavior or INTERVENE, you should feel one buzz; for each hold (≥1 s) you should feel two buzzes. This can be tested with the real hardware in hand. If the motor is small, the buzz can be subtle – you may adjust the duration or even voltage (if using a bench supply) to get a clear feeling. The motor feedback combined with the OLED should give the user a confirmation they feel and see for each input, which is important for eyes-off use in stressful moments.

With both OLED and haptic feedback now active, the device provides multimodal confirmation for every action. This was a design must-have to ensure the caregiver doesn’t have to second-guess whether a press “went through” while they are managing a crisis.

Lesson 18: Implementing Target Tagging Logic

Duration: ~1 hour

Add the logic for the side target buttons (ME, SIB, OTHER) that “tag” the next behavior event. The PRD specifies how these work: pressing one of these doesn’t create a log entry by itself, it just sets an internal pending_target state. The next behavior event logged should include that target, then the pending_target resets automatically.

Implement this by watching for target button presses in your input handling. If one of the target buttons is pressed, store its identifier (e.g., pending_target = "SIB" if the SIB button was pressed). You might also give some immediate minor feedback for arming a target – perhaps a subtle OLED indicator like highlighting that target on screen, or maybe no haptic (per the spec, no buzz on target press). The user will generally know they pressed it.

Modify the event logging function to incorporate the target: when logging an event, check if pending_target is set and the event is a behavior (VERBAL, PHYSICAL, etc.). If so, include the target value in the CSV for that event and then clear pending_target back to “none”. If the event is REGULATED or an INTERVENE action, do not apply any target (those events are always about the caregiver or resolution, not directed at someone). In those cases, also clear any pending target since the incident context changed, or simply keep it (depending on spec – likely better to clear to avoid accidentally carrying over a target through an entire incident).

Test the target tagging feature thoroughly:

Press a target (say SIB), then press a behavior like PHYSICAL. The first PHYSICAL event logged should show target “SIB”. Then press another PHYSICAL without hitting a target again – this one should log with target blank (no target) since the pending target was already consumed.

Repeat for ME and OTHER to ensure each works. Also ensure if you press a target and then press a REGULATED or INTERVENE, the target is ignored for those (and likely cleared). For example, pressing SIB then pressing the INTERVENE button should result in an ATTEMPT_SUPPORT log with no target (and the SIB tag should not linger indefinitely).

This feature adds important context to the data (who the behavior was directed at) and is a good exercise in managing state across multiple button inputs. After this, your event logs will contain the extra column data when appropriate, which aligns with the intended usage of the device.

Milestone: Target tagging is functional. The next logged behavior after a target button press gets tagged with “ME”, “SIB”, or “OTHER” as appropriate, and the system then clears the tag. This meets the requirement of capturing who the behavior was aimed at without requiring separate logging steps for the user.

Lesson 19: Integration Test – Full System on the Bench

Duration: ~2 hours

At this stage, all major subsystems (inputs, outputs, logging, audio, state management) have been developed individually. Now it’s crucial to test the entire system working together on the bench, before packing it into an enclosure.

Assemble all components if you haven’t already: the ESP32 board with the OLED, the SD card module, the I2S mic, the vibration motor, and all the buttons should all be connected simultaneously. Use the printed pin map and wiring diagram you prepared to ensure everything is connected to the right pins. This may be on a single breadboard or a proto-board. Keep the wiring tidy to avoid shorts or confusion (twist wires, use color coding if possible).

Run a comprehensive integration test guided by the Test Plan:

Button functionality: One by one, test each front button (VERBAL, PHYSICAL, etc., including INTERVENE) with short taps. Verify: you feel one haptic buzz, the OLED shows the correct label (and perhaps target/flag if any), and a new log entry appears in events.csv with the correct behavior name. Then test holding each of those buttons ≥1 s. Verify: you feel two buzzes, the OLED indicates the flag (e.g., “threat” for VERBAL hold), and the CSV entry has the correct flag field. Ensure none of these inputs are missed or misclassified.

Target tagging: Test the side buttons by doing the sequence: press SIB, then tap a behavior (e.g. PHYSICAL), then tap the same behavior again without a target. Confirm the first event got target=SIB and the second did not. Also confirm that pressing a target button doesn’t itself produce any log line (the CSV should have no entry just for “SIB pressed”) and ideally no buzz either. Do this for ME and OTHER as well.

Incident lifecycle: Simulate a full incident. For example: tap PHYSICAL to start an incident (check that “REC” appears on OLED and an audio file is created), then press a mix of events: maybe press ME then hold VERBAL (to simulate a threat at the caregiver), tap INTERVENE (to log an ATTEMPT_SUPPORT), tap REGULATED (to simulate de-escalation), then stop pressing and let the cooldown run out. Verify the following: the audio continued throughout and stopped after ~5 minutes of no input, an INCIDENT_END was logged, and the OLED “REC” turned off at the end. All events in that sequence should have the same incident filename in the CSV and that filename should correspond to the audio file recorded.

After the incident, remove the SD card and inspect the files on a computer. There should be one audio WAV file for the incident and the CSV should show a sequence of events matching what you did (with proper timestamps, targets, flags). Check that the CSV is properly formatted and in order. Play back the WAV and confirm it has audio spanning the incident (you should hear the moments when you pressed buttons, like any verbal call-outs you did). This is effectively an end-to-end test of the device’s purpose.

If any issues arise (and it’s normal if they do): use systematic troubleshooting. For example, if a certain button didn’t log, check its wiring and pin definition; if the audio file is empty or corrupted, check your I2S buffer handling or file writing logic. Refer back to the Test Plan’s acceptance criteria to see where it might be failing, and iterate on the code/hardware to fix it. This is a great learning moment on debugging a complex system: tackle one subsystem at a time to isolate the problem.

This integration test is a major milestone. At its completion, you should have a working prototype on the bench that meets the functional requirements of the Stella Logger. It’s essentially the device without a case. Take a moment to appreciate how far the project has come!

Lesson 20: Enclosure Design and Hardware Assembly

Duration: ~2 hours (could span multiple sessions for fabrication)

Discuss the mechanical design of the device. The hardware spec provides guidelines for the enclosure and layout (dimensions, ergonomics). Review these targets: roughly 12 cm tall, 5 cm wide, 2 cm thick, lightweight, one-handed operation, and a neutral appearance (it should not look like a weapon or flashy gadget).

Plan the front panel layout for the seven front buttons. According to the design, six behavior buttons are arranged in a 2×3 grid, and the INTERVENE button is distinct (e.g., centered below the grid). The OLED display should be near the top for visibility. Mark the positions for these on paper or using a template. Ensure the INTERVENE button stands out by feel (maybe a different cap or shape) as per spec.

Plan the side panel for the three target buttons. They should be a vertical stack on the side (likely the right side for a right-handed thumb to press). Space them so they can be distinguished by touch. Possibly label or texture them differently (or plan to do so).

Determine how you will fabricate or obtain the enclosure. Options: a pre-made plastic project box of suitable size (which you can drill/cut), a 3D printed custom case, or even a laser-cut acrylic box. The Elegoo kit components (like buttons) might be mounted on perfboard that then goes into the case. The hardware spec suggests using an ABS handheld enclosure around Hammond 1553B size – if you have something similar, that could work.

If drilling by hand, create a drill guide (even just a paper printout taped to the box) to mark holes for the buttons, display, USB port, and SD card slot. Accuracy ensures everything fits well. Use a small pilot drill and step up sizes for clean button holes. Be mindful of aligning the OLED window if you want one.

Mount components securely: screw down the PCB if the enclosure has standoffs, or use spacers and epoxy. For buttons, if they are loose, consider a custom front plate or hot glue to hold them in place aligned with the holes. The OLED can be held with screws (if the breakout has holes) or with a snug panel cutout.

Address strain relief: the LiPo battery’s leads should be secured so they can’t be yanked off the board (a bit of hot glue or a cable tie anchor can do this). Similarly, ensure the lanyard anchor (if any) is attached to the case itself, not to the circuit board or battery. The hardware spec emphasizes this to prevent internal damage when the device is worn. If your case doesn’t have a lanyard loop, you may improvise one (e.g., a small eyelet screw in the plastic).

Keep the microphone’s placement in mind: there should be a small hole or grille in the case for the mic, and it should not be obstructed by the user’s hand or the lanyard. Make sure the mic hole is not where it could rub on clothing too much (which could create noise).

Once everything is in the enclosure, do a basic functionality re-test (Lesson 19’s tests) to ensure nothing got disconnected in the process of assembly. Sometimes a wire can slip off or a solder joint can break while fitting components in a case. It’s easier to fix before everything is fully closed up.

Lesson 21: Final Testing and Validation

Duration: ~1.5 hours

Perform a full validation test on the assembled device, just as a user would use it, to ensure it meets all design requirements. Follow the official Test & Validation Plan as a guide:

Button functionality (Confirmation): Test each front button tap and hold again on the assembled device. You should feel the correct haptic feedback and see the OLED confirmation for each input. Verify the log entries on the SD card correspond exactly (correct behavior names, flags for holds, etc.). There should be 0 missed presses here – every press yields a log.

Target tagging: Press each target (ME, SIB, OTHER) followed by a behavior, and ensure the tag applies only to the next event and then clears. Check the CSV after a few tries to be sure it’s logging correctly (target field present where expected, blank where not).

Incident flow: Do a scenario test: e.g., Start idle, press PHYSICAL (incident starts), then maybe press SIB + PHYSICAL, then INTERVENE (tap and hold), then REGULATED. Stop and let it cool down. This should produce a complete incident log and audio. Confirm INCIDENT_END was added after ~5 minutes of inactivity. Ensure the OLED showed “REC” during the active/cooldown and turned off after. The test plan’s Scenario Run (section H) outlines a similar end-to-end sequence – you can mimic that and see if the outcome matches expected.

Data integrity: Remove the microSD and open events.csv. Verify it’s readable and that the entries make sense and are in order. Check that incident file names in the CSV match actual files on the card. Play the audio files on a computer: each should play from start to finish without errors, and you should roughly hear events corresponding to the log (e.g., if you said something at a REGULATED moment, you find that spot in audio). This ensures no file corruption and correct synchronization.

Power interruption test: While an incident is active (REC on), turn the device off abruptly (if you have a power switch or by pulling the USB if running on USB power). Then turn it back on. The device should reboot into IDLE state with no incident active (since it was cut) and the CSV should still contain all events up to the cut (maybe missing an INCIDENT_END for that interrupted incident, which is acceptable). This tests robustness against unexpected shutdown.

Usability under stress: Evaluate the device as if in real use. Try using it with one hand without looking: wear it on the lanyard, simulate walking or moving, press a target and a front button by feel. It should be doable thanks to the physical layout. Press buttons in quick succession (like two or three different events within 2 seconds) and confirm all are logged and feedback given (the system shouldn’t freeze or choke). Glance at the OLED briefly when pressing an event – ensure you can catch the confirmation text in the ~2 s it’s displayed and that the text is readable. This addresses real-world operation where attention is divided.

Survivability tests: If possible, gently perform the drop test – with the device powered and logging, drop it from chest height onto a soft surface like carpet. It should continue running and not reset. Then do a lanyard tug test – wear it and give the lanyard a firm pull as might happen if a child grabs it. The device should suffer no internal disconnections (it should still log an event after the tug). These tests verify the robustness of your assembly (no loose wires, battery secure, etc.).

Appearance check: Look at the assembled device and consider if it meets the “non-escalating look” requirement. There should be no flashy lights visible externally (the OLED is the only indicator and it’s relatively subtle). The form should be neutral – at this point, any cosmetic adjustments (like smoothing out sharp edges, or coloring it in a neutral tone) can be considered.

Any test that does not pass is a cue to troubleshoot and improve (e.g., if a button missed a press in the drop test, maybe a loose connection needs soldering). Refer to the acceptance thresholds in the test plan to know what “pass” looks like (e.g., 0 missed presses out of dozens tested, all audio files play correctly). Make adjustments, then re-run tests as needed.

Milestone: All validation tests are passed (or acceptable issues resolved) – the device is officially a working prototype ready for real-world trials (Go for deployment). This is the successful culmination of the project build.

Lesson 22: Troubleshooting and Support Skills

Duration: ~1 hour

Reflect on the development process and discuss troubleshooting strategies that were useful, as well as additional ones for future issues:

Firmware debugging: Using serial printouts was likely a primary debugging tool. Reinforce how printing key state changes or sensor values can illuminate what the code is doing. Mention that one can also use the onboard LED or even the OLED to display debug info if serial isn’t accessible in a field scenario.

Hardware debugging: Emphasize using a multimeter (from the kit) to check continuity and voltages. For instance, if a button isn’t working, check that the GPIO pin sees 3.3 V when not pressed and 0 V when pressed. If not, there’s a wiring or solder issue. If the device isn’t powering on, verify the battery voltage and that the ESP32’s regulator isn’t in a fault state.

List common issues and fixes: e.g., SD card errors (often fixed by checking wiring of MISO/MOSI/CS or ensuring the card is FAT32), OLED not displaying (check I2C address or that SDA/SCL aren’t swapped), mic recording noise (ensure the mic isn’t next to the motor or digital lines, maybe add a small delay to reduce CPU noise, or lower gain if possible).

Discuss the importance of isolating subsystems when troubleshooting. If audio recording isn’t working, create a minimal program that just tests the mic and SD writing separate from the rest, to pinpoint if it’s a library issue or an integration timing issue.

Highlight the value of the Test Plan as a continual reference. It’s not only for final testing – one can use it during development to test features as they’re implemented. It also provides a clear definition of done for each aspect. Encourage the student to use similar checklists for future projects to ensure reliability.

Ensure the learner feels confident in supporting the device they built. For example, if after some use a button starts acting flaky, they should know how to open the case and inspect or replace it. If the battery seems to not hold charge, they should know how to test it or swap it out safely. This course was not just about building one gadget, but also about instilling good engineering practices for maintenance and iteration.

Lesson 23: Project Wrap-Up and Next Steps

Duration: ~0.5 hour

Recap the entire project and congratulate the learner on building a complex, functional embedded system from scratch. Summarize the knowledge and skills acquired: from basic electronics (circuits, pull-ups, debouncing) to firmware development (Arduino coding, using I2C/SPI/I2S protocols) to higher-level concepts (state machines, data logging) and finally to testing and iterating a design for real-world use.

Revisit the rationale behind key design choices now that the student has firsthand experience:

Why ESP32? – for its performance and integrated features (we utilized its Wi-Fi/Bluetooth-capable MCU mainly for I2S audio and file system in this project, plus the convenience of a dev board with battery charging).

Why physical buttons? – in a crisis scenario, tangible buttons arranged by function allow eyes-off operation and immediate action, which a touchscreen or sequence-based input couldn’t provide.

Why a digital MEMS microphone and microSD? – a digital mic (I2S) simplified the audio signal chain (no need for analog ADC tuning) and provided suitable quality. MicroSD storage offers large capacity and easy data transfer by just handing the card or plugging into a PC, meeting the requirement of easy data export for the caregiver.

Why haptic feedback over audible feedback? – a buzzing motor is private and won’t escalate the situation, aligning with the “non-escalating” design goal. An audible beep or loud sound could have aggravated the child or drawn unwanted attention.

Power design choices: using a rechargeable LiPo keeps the device lightweight and always ready (as opposed to replaceable batteries), and the built-in charging circuit on the Feather simplified safe charging. We also adhered strictly to safety practices (no dangerous power switches, proper strain relief) to ensure reliability under rough use.

Encourage the learner to think of future enhancements or versions:

Adding a real-time clock (RTC) module to timestamp events with actual date/time (so logs have absolute times).

Utilizing the ESP32’s Bluetooth or Wi-Fi to sync data to a phone or cloud in real time (though note, this adds complexity and was deliberately left out in v1 for reliability, but it’s a great next project if they wish to attempt it).

Designing a custom PCB for the circuit to reduce wiring and improve robustness. This could be a whole learning project in PCB design and would make the device slimmer and more durable.

Refining the enclosure: perhaps 3D printing a more ergonomic case, adding waterproofing or shockproofing, or simply making it look more polished. Cosmetics matter if the device is to be used in public – a professional-looking device can help the user feel more confident using it.

Stretch features from the PRD like a battery percentage display, incident segmentation if one lasts too long, or minor haptic feedback when arming a target (a tiny “tick”) could be interesting tweaks to explore.

Lastly, reflect on the real-world impact of the project. The Stella Behavior Logger is intended as a tool to aid in extremely challenging situations – by building it, the learner not only gained technical skills but also created something that aligns with values of safety and empathy (documenting not just “bad moments” but also repair and caregiver interventions). This is a meaningful achievement. Encourage the learner to document their build (for their portfolio or others who might want to build it) and feel proud of the new skills they’ve acquired.