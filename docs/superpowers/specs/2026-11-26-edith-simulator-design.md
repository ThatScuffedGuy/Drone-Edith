# Edith Simulator — Design

Date: 2026-11-26
Status: Draft for approval

## 1. Intent

The owner dreamed of a palm-sized drone, "Edith", that rides on their back
(like the little spider drone on Spider-Man's chest that detaches, scouts,
talks, and snaps back). In the dream they said: *"Edith, search the building
and map it out for me, and let me know if you find the hostages."* Edith flew
off, searched on its own, and reported back.

They want this for real eventually — both for that search-and-report use and
as a **pet / sidekick they can talk to and give commands**. The first step,
chosen by the owner, is to build **Edith's brain in a simulator** so it can
be developed and tested for free and safely, then moved onto a real drone
later.

What makes Edith special, in the owner's words and picture:

- Voice-activated, Jarvis-style. You talk, it answers, with a personality.
- It is part of your outfit: it lives in a dock on your back, detaches,
  flies off, and comes back and parks itself.
- It works **on its own until the job is done or you call it back**.
- It searches, maps, and finds things: people, hostages, hazards, threats
  (e.g. "if it sees a gun, it alerts me").
- It is quiet and intelligent; you can give it many different commands.
- It is linked to a **wrist pad** (a watch/wrist screen) that shows what it
  is doing and what it found.

## 2. Success looks like

The dream scene, playable on a laptop:

1. The owner stands their avatar at the entrance of an abandoned warehouse,
   Edith docked on their back.
2. They say out loud: "Edith, search the building and map it out for me.
   Let me know if you find the hostages."
3. Edith answers in character, detaches, and flies in.
4. On the wrist pad (their phone, or an on-screen panel), the map draws
   itself room by room, with Edith's position.
5. Edith interrupts by voice and on the wrist pad when it sees something:
   "Careful — armed person, east hallway." / "Found two hostages, back
   office. They look restrained." / "Hazard: hole in the floor, storage
   room."
6. When the building is fully searched (or the owner says "Edith, come
   back"), Edith reports a summary, flies back, and docks on their back.
7. The owner can ask follow-ups: "How many people did you find?" "Where
   was the gun?" "Go back and watch the hostages."

No keyboard needed except to walk the avatar around.

## 3. Goals

- **G1 — Talk to Edith naturally.** Wake word "Edith", free-form speech,
  spoken replies with a consistent personality. No memorized command list.
- **G2 — Missions that run on their own.** Edith keeps working a mission
  until it decides it is finished, is recalled, or must return (low
  battery). It never needs to be steered.
- **G3 — Autonomous mapping.** Edith explores unknown space and builds a
  map as it goes (it does not get to "peek" at the level layout).
- **G4 — Find and classify things.** People (unknown / hostage / armed),
  weapons, hazards. Each with a location and a confidence.
- **G5 — Alerts that match the danger.** Threats interrupt immediately;
  hostages/people next; hazards after. Spoken + wrist pad + map marker.
- **G6 — Wrist pad.** Live map, Edith's position and status (mission,
  battery), alert feed, Edith's camera view, and buttons for the most
  important actions (recall, pause/hold, acknowledge alert).
- **G7 — Launch from and dock on the back.** Edith detaches from the
  avatar's back dock, and returns and lands precisely on it.
- **G8 — Pet mode.** When not on a mission Edith can ride on your back,
  hover at your shoulder and follow you, chat, and come when called.
- **G9 — Ready for a real drone.** Edith's brain talks to its body through
  one narrow interface, so a real-drone body can replace the simulated one
  later without rewriting the brain.

## 4. Non-goals (for this first version)

- Real hardware, real flight controllers, real radio links.
- Real camera-based AI (e.g. a neural network spotting guns in images).
  Perception is simulated (see §7.5).
- Multi-floor buildings, stairs, or outdoor scenes. One floor only.
- Docking while the avatar is moving. The owner stands still to dock;
  Edith asks them to hold still if they are moving.
- Realistic aerodynamics. Flight is smooth and plausible, not physics-exact.
- Multiplayer, multiple drones, or enemies that act/move (the scene is
  static except for Edith and the avatar).
- A dedicated smartwatch app. The wrist pad is a web page.
- Copying the look of any movie drone or character. Edith's look is our own.

## 5. Anti-goals (would make this a failure even if it "works")

- Edith feels like a menu or a remote control instead of a partner you talk
  to.
- Edith stops working, gets stuck, or goes silent without telling you why.
- Edith cheats: it knows the layout or where the hostages are before it has
  actually seen them.
- Edith states something as certain when it is not (e.g. "gun" when it only
  glimpsed something at the edge of view). It should say "possible weapon"
  and, where safe, take a closer look.
- Edith flies right up to an armed person. It keeps its distance from
  threats.
- It only works with a perfect internet connection. Without the AI service,
  basic commands must still work.

## 6. Constraints

- Runs on an ordinary laptop in **Chrome or Edge** (needed for built-in
  speech recognition). Needs a microphone and speakers.
- The wrist pad runs in a phone browser on the same Wi‑Fi, or as an
  on-screen panel on the laptop when no phone is used.
- Uses Claude (Anthropic API) for language understanding and personality.
  Needs an API key and internet; costs a few cents per mission.
- The owner is not expected to be a programmer. Starting the simulator
  must be one command (or a double-click script) plus pasting an API key
  once.

## 7. Design

### 7.1 Overall shape

```
 ┌──────────────── Laptop browser ────────────────┐      ┌── Phone browser ──┐
 │  3D World (building, avatar, Edith)            │      │   Wrist Pad       │
 │        ▲ sensors        │ motion               │      │  map / alerts /   │
 │        │                ▼                      │      │  camera / buttons │
 │  ┌──────── DroneBody interface ─────────┐      │      └────────▲──────────┘
 │  │ SimBody (now)  /  RealBody (later)   │      │               │ live updates
 │  └──────────────────▲───────────────────┘      │               │
 │                     │                          │      ┌────────┴─────────┐
 │  EDITH BRAIN: Mapper · Planner · Perception →  │◄────►│  Local server    │
 │  Mission runner · Alerts · Memory              │      │  (relay + Claude │
 │  Voice in (speech→text) / out (text→speech)    │      │   API key)       │
 └────────────────────────────────────────────────┘      └──────────────────┘
```

- **Laptop browser page:** 3D world, Edith's brain, voice in/out.
- **Local server:** serves the pages, holds the Claude API key (never sent
  to the browser), calls Claude, and relays live state to the wrist pad.
- **Wrist pad page:** phone-sized UI; also embeddable as an overlay on the
  laptop.

Tech: TypeScript everywhere; Three.js for 3D; Vite for the web app; Node
server with WebSockets; Vitest for tests.

### 7.2 Brain / body split (G9)

Edith's brain only talks to its body through a `DroneBody` interface:

- commands: take off, fly to a point at a height/heading, hover, rotate,
  land on a target, emergency stop
- readings: position/heading estimate, battery, range readings (a ring of
  simulated distance sensors), camera "detections" (§7.5), docked/flying
  state

`SimBody` implements this in the 3D world. A later `RealBody` (e.g. a small
programmable drone) implements the same interface. Nothing above the
interface may reach into the 3D scene directly.

### 7.3 The world

- A hand-built **abandoned warehouse** scenario matching the dream: entrance,
  hallways, offices, storage, a back office. Placed in it: 2 hostages
  (restrained, seated), 1–2 armed people, 1 unarmed bystander, a loose
  weapon on a table, and hazards (hole in the floor, exposed live wires,
  leaking gas/fire).
- Scenarios are data files (layout + placed objects) so more can be added
  without code changes. Placements can optionally be randomized each run so
  the owner can't memorize them.
- Dim, moody lighting; the visuals are simple low-poly shapes, readable
  rather than realistic.
- The avatar is a simple third-person character walked with WASD / arrow
  keys and mouse to look. The back dock is a visible plate between the
  shoulders.
- Edith is a small quadcopter model with spinning props and a status light
  (colour shows mode: docked, searching, alert, returning).

### 7.4 Mapping and moving

- **Map:** a 2D top-down grid (unknown / free / wall), built only from
  Edith's simulated range sensors as it flies. Rooms get simple labels as
  they are discovered ("east hallway", "back office") from position on the
  map, so both Edith and the owner can refer to places.
- **Exploring:** Edith picks the nearest reachable edge between known and
  unknown space, flies there, and repeats until no reachable unknown space
  is left. That is when a search is "done".
- **Path planning:** shortest safe path on the known map, keeping a margin
  from walls and a larger margin from known threats and hazards.
- **Battery:** simulated, ~10 minutes of flight. Edith keeps enough reserve
  to fly home; when it hits that reserve it tells the owner and returns,
  even mid-mission.

### 7.5 Seeing things (simulated perception)

Edith does not read the answers from the level. Its "camera" is simulated
honestly:

- Things are only seen if inside the camera's field of view, within range,
  and not blocked by walls (line-of-sight check).
- Each detection has a **confidence** that depends on distance, viewing
  angle, and lighting. Far/edge glimpses are low-confidence.
- Low-confidence findings are reported as "possible …". For people and
  objects that are **not** threats, Edith may move closer to confirm. For a
  possible weapon/armed person, Edith confirms from a distance (changing
  angle, not closing in).
- The camera view on the wrist pad is a small render from Edith's point of
  view, with boxes and labels on what it detected.

Categories: person (unknown / hostage / armed), weapon, hazard (hole, live
wires, gas/fire).

### 7.6 Missions (G2)

Edith runs one mission at a time, plus pet behaviours when idle. Missions:

| Mission | Finishes when |
|---|---|
| Search & map (optionally "look for X") | whole reachable floor explored |
| Find X (e.g. "find the hostages") | whole floor explored; X are reported the moment they're found, but Edith keeps going unless told "stop when you find them" |
| Go to a place ("check the back office") | place reached and looked around |
| Watch / guard a place or finding ("watch the hostages") | recalled |
| Return & dock | docked |
| Follow me / stay at shoulder (pet mode) | told otherwise |

Rules:

- Any mission can be interrupted by "Edith, come back", "stop/hold", or a
  new mission. A new mission replaces the old one after Edith confirms
  ("Switching to …").
- On finishing, Edith gives a short spoken summary and returns to dock
  unless the mission says otherwise.
- If stuck (no path, lost), Edith says so and either returns or holds and
  asks for instructions. It is never silent.

### 7.7 Talking to Edith (G1, G8)

- **Listening:** browser speech recognition, always listening for the wake
  word "Edith". Also a push-to-talk key (Space) and a mic button on the
  laptop overlay as a fallback.
- **Understanding:** the spoken sentence, plus a compact summary of Edith's
  current state (mission, battery, findings, known places), goes to Claude.
  Claude answers with (a) what Edith says and (b) zero or more actions from
  a fixed list (start mission, recall, hold, set alert preferences, answer
  from memory). **Claude chooses actions; the mission code carries them
  out.** Claude never flies the drone directly.
- **Personality:** loyal, calm under pressure, a bit witty in pet mode,
  short and clipped during danger. Defined in one editable prompt file.
  Edith remembers what it found during the session and can answer
  questions about it.
- **Speaking:** browser text-to-speech with a chosen voice. Alerts preempt
  whatever Edith is saying.
- **Offline fallback:** if there is no API key or no internet, a simple
  keyword matcher handles the core commands (search, find hostages, come
  back, stop, follow me, status) with canned replies. Edith says it is in
  "basic mode".
- **Model:** a fast Claude model by default for quick replies, configurable
  in a settings file.

### 7.8 Alerts (G5)

Priority, highest first:

1. Weapon / armed person
2. Hostage
3. Other person
4. Hazard

Each alert: spoken immediately (threats interrupt speech), shown on the
wrist pad feed with location and confidence, pinned on the map, and the
phone vibrates for priority 1–2 where the phone supports it. Duplicate
sightings of the same thing update the existing alert instead of making a
new one. The owner can acknowledge alerts from the wrist pad.

### 7.9 Wrist pad (G6)

Phone-sized, dark, glanceable:

```
┌──────────────────────────┐
│ EDITH  ● SEARCHING  ▮▮▮▯ │  mode + battery
├──────────────────────────┤
│   [ live top-down map ]  │  Edith ▲, you ●, pins
│                          │  (! threat, H hostage)
├──────────────────────────┤
│ ! Armed person  E hall 90%│  alert feed, newest first
│ H Hostages x2  back office│
├──────────────────────────┤
│ [CAMERA] [HOLD] [RECALL] │  buttons
└──────────────────────────┘
```

- Opened by scanning a QR code shown on the laptop (same Wi‑Fi).
- The same panel can be shown as an overlay on the laptop screen.
- Camera button swaps the map for Edith's live view.

### 7.10 Launch and docking (G7)

- **Launch:** Edith releases from the back plate, rises above head height,
  and turns toward its first goal.
- **Return:** Edith flies home along the known map to just behind the
  avatar, matches the avatar's facing, then descends slowly onto the plate,
  steering on the plate's marker (stands in for the visual marker + magnet
  catch a real dock would use). Snap-lock when within tolerance.
- If the avatar is moving, Edith hovers behind and says "Hold still, I'm
  coming in." If docking fails, it backs off and retries (up to 3 times),
  then hovers at the shoulder and says so.

## 8. Approaches considered

| Approach | Trade-offs | Decision |
|---|---|---|
| **A. Browser 3D simulator** (Three.js, TypeScript) | Runs on any laptop; quick to see results; easy to share; wrist pad is just another web page. Simplified physics; perception is simulated rather than real camera AI. | **Chosen by the owner.** Fastest path to a living, talking Edith; the brain/body split keeps the route to a real drone open. |
| B. ROS 2 + Gazebo robotics simulator | Same stack as real research drones; realistic physics and real camera AI. Needs Linux, a good GPU, heavy setup, steep learning curve, plain visuals. | Not chosen — too heavy for a first step. |
| C. Game engine (Unity/Unreal) | Cinematic visuals realistic enough for real vision AI. Big install, slow to build, the main drone sim plugin is no longer maintained. | Not chosen — cost/effort outweighs gains now. |

Earlier choice (also the owner's): **simulator first**, then a real small
programmable drone, then possibly a custom build.

## 9. Decided by the owner

- Simulator first (approach A).
- Voice activation with the name "Edith"; Jarvis-like conversation.
- Pet-like companion as well as a search tool.
- Stays on a command until it knows it is finished or is told to come back.
- Detects people, hostages, hazards, threats (e.g. guns) and alerts.
- Wrist pad / watch display linked to Edith.
- Lives on and docks to the owner's back.

## 10. Calls made by the designer (owner can override)

- Brain runs off-drone (on the laptop now; a carried phone/computer later).
- Wrist pad = phone web page (plus laptop overlay), not a smartwatch app.
- Docking only while standing still in v1.
- Claude for language/personality; deterministic code for flying and
  missions; offline keyword fallback.
- Single-floor warehouse scenario first; scenarios as data files.
- Find-missions keep searching the whole floor after the first find unless
  told otherwise.
- Edith confirms possible threats from a distance, never by closing in.
- Battery ~10 simulated minutes with automatic return on reserve.
- Chrome/Edge required for speech recognition.

## 11. Left to the builder

- Exact file/folder layout, module boundaries below the `DroneBody`
  interface, and state management.
- Map grid resolution, sensor count/ranges, confidence formula, exploration
  tie-breaking — tune so a full warehouse search takes ~2–4 real minutes.
- Visual style details, colours, Edith's model, the chosen TTS voice.
- Exact Claude model name and prompt wording (within §7.7).
- How the server is started (npm script, plus a double-click script for
  Windows/macOS).

## 12. Testing

- Unit tests (Vitest) for: map building from range readings, path planning,
  exploration "done" detection, perception visibility/confidence, mission
  state transitions (start, interrupt, recall, low battery, stuck), alert
  de-duplication and priority.
- The Claude layer is tested with recorded example replies, so tests run
  without an API key or internet; the offline fallback is tested directly.
- A scripted end-to-end run of the warehouse scenario (text commands
  instead of voice) that checks: all hostages, weapons and hazards are found
  and alerted, the map covers the reachable floor, and Edith docks at the
  end.
