```
DESIGN_DOC
game_name: scent-trail
display_name: Scent Trail
```

---

# SCENT TRAIL
### *You can see exactly where it is. That's the problem.*

---

## WHAT MAKES THIS GAME THIS GAME

Before anything else: the one irreducible thing.

**This game is about knowing.** You know where the beast is. You know its cone. You know how long the bait holds it. You do the arithmetic. You are still wrong. That delta — between your knowledge and your survival — is every second of this game. It is not a horror game. It is a precision game with dread as its feedback loop.

The beast is never a surprise. The beast is a moving wall with a timer on it. The skill is measuring.

---

## IDENTITY

**Game name:** Scent Trail  
**Tagline:** *Drop it. Watch it turn. Run.*  
**What is the player:** A small human figure, gender-neutral, hunched slightly — someone who knows what's in here with them. Not a hero. A survivor trying not to breathe too loudly.

**World feel:**  
Ancient stone corridors, damp, lightless except for the faint amber pulse in the player's hand and the faint red smear of beast eye-shine in the dark. The scale is wrong — the beast is so large it has to lower its head to fit the corridor, and when it chews the bait you can hear it from three rooms away.

**Emotional experience:** *Surgical dread.*

**Reference games:**

| Game | DNA extracted |
|---|---|
| **Inside (Playdead)** | Silhouette-first visual grammar. The monster is scarier for being faceless. Restraint as a design principle — what is withheld is more terrifying than what is shown. No UI contamination of the world. |
| **Gran Turismo (series)** | The corner you can take flat-out only after 20 attempts. The skill is not reaction time — it is *measurement*. You learn the exact braking point through repetition, not intuition. Scent radius works identically. |
| **Metal Slug** | Instant threat-reading. Every enemy's reach, speed, and timing is visible and learnable. The game is hard because the information is clear and you still miscalculate, not because information is hidden. The attention cone is Metal Slug's bullet arc. |

---

## VISUAL SPEC

### Color Palette

| Role | Hex | Usage |
|---|---|---|
| Background / Cave black | `#0D0B08` | All background fill, void, deep shadow |
| Warm stone | `#1C1812` | Stone floor, wall surfaces, mid-ground detail |
| Stone highlight | `#2E2820` | Wall edge catches, floor grain |
| Player silhouette | `#1A1510` | Player body fill — nearly black, slightly warmer than void |
| Player rim light | `#3D3228` | 1px inner rim on player left edge only — implies single light source from far door |
| Bait amber (dim) | `#6B4F1A` | Bait resting state |
| Bait amber (pulse) | `#C8A96E` | Bait held / just-dropped state, animation keyframe bright |
| Bait amber (core) | `#F0D49A` | Bait bloom core at peak pulse |
| Beast silhouette | `#0A0806` | Beast body — darker than void, a moving hole |
| Beast eye-shine | `#8B1A1A` | Two small fixed-position rectangles on beast head, always visible |
| Beast eye-shine hot | `#C42222` | Eye-shine when beast is in ALERTED state (head turning toward player) |
| Scent ring | `#C8A96E` at 15% opacity | Expanding ring from dropped bait |
| Footstep ripple | `#A0A0A0` at 10% opacity | Expanding ring from player feet on stone floor (Level 3+) |
| Exit door glow | `#E8E0C0` | The lit door — warmest, brightest thing in scene |
| Exit door core | `#FFFFFF` at 60% | Inner glow of exit — the goal is always visible |

### Bloom
- **Yes**
- Strength: `0.55`
- Threshold: `0.60` (only bait amber, exit glow, and beast eye-shine bloom — stone surfaces never bloom)
- Bloom applied as post-process additive pass. Bait bloom radius: 18px at resting pulse, 32px at drop impact frame.

### Vignette
- **Yes**
- Strength: `0.70` (heavy — corners are near-black, center is visible)
- Shape: oval, not square
- The vignette compresses the readable world to the center strip where the action happens.

### Camera
- **Type:** Fixed side-scroll, orthographic
- **Position:** Eye-height — the camera sits at approximately 1.4 units height in a 3-unit-tall corridor. Player eye level. The ceiling is cut off. You never see above the beast's shoulders.
- **Horizontal:** Camera scrolls on X only, locked to player with 60px lookahead in direction of movement. When player is stationary, camera drifts 20px toward the nearest beast over 2 seconds (slow pull — builds dread without the player moving).
- **No vertical scroll.** The world is one level.
- **Viewport:** 1080×540px (2:1 ratio). Pixel density: sprites at 2px per world unit base.

### Player Silhouette
> **Crouched, cautious, carrying something precious.**

Specific breakdown:
- Height: 28px tall (beast is 84px tall — 3x ratio, deliberate)
- Body: hunched posture — spine at 20° forward lean
- Arm: one arm slightly extended forward (holding bait) — creates visual hook for "this thing is droppable"
- Color: `#1A1510` fill, `#3D3228` 1px right edge rim
- Run animation: 6 frames, no bounce — flat, urgent, not heroic
- Idle animation: weight shift every 2.4 seconds, 2-frame micro-bob. Bait hand trembles with bait pulse.

---

## SOUND SPEC

### Music
- **BPM:** 42
- **Description:** Single drone piece. Built from: one bowed contrabass pedal tone (open A, sustained), one sub-bass sine wave at 38Hz cycling at 42BPM with very slight LFO wobble (±2Hz, 0.3Hz rate), and sparse single piano notes (cold, reverbed to 4 seconds) that occur on no fixed interval — every 8–22 seconds, random. No melody. No rhythm in the conventional sense — the 42BPM is a heartbeat not a groove.
- **The music is almost nothing.** The absence is part of it.

### Music Responds To

| State | Change |
|---|---|
| Beast IDLE (not chewing, sweeping normally) | Base drone only, sub-bass at 30% volume |
| Beast ALERTED (head turning toward bait or player) | Sub-bass swells to 70% over 0.8 seconds |
| Beast within 3 tiles of player | Sub-bass at 100%, additional 60Hz sine layer added at 40% volume — two bass tones now |
| Beast within 1 tile (near-death) | High-frequency silence cut: piano notes stop entirely. Only bass. |
| Player holding last bait | Piano note interval tightens: fires every 6–10 seconds instead of 8–22 |
| Player bait-depleted (0 bait remaining) | Sub-bass adds subtle distortion (3% overdrive) — the silence feels wrong |
| Stage clear (player reaches exit) | All audio fades to silence over 1.2 seconds. Door opens. Silence. Not triumph. Relief. |
| Death | Immediate audio cut to silence for 0.5 seconds, then low 80Hz thud (see SFX below), then silence for respawn |

### Sound Effects (6)

**1. Beast footstep — stone impact**
- Trigger: Each beast step. Step interval varies by level (see Level Design).
- Tone.js: `MembraneSynth` with `pitchDecay: 0.08`, `octaves: 3`, base pitch `C1` (32.7Hz). Add `Distortion` effect at `0.15` wet. Add `Reverb` with `decay: 1.8s`, `wet: 0.4`. Output through `Compressor` with `-24dB` threshold, `8:1` ratio to give it chest-feel without clipping.
- Character: Wet, heavy, felt more than heard. Should rattle the sub. This is the game's heartbeat.

**2. Bait drop — impact**
- Trigger: Bait hits floor.
- Tone.js: `MetalSynth` with `frequency: 180`, `envelope: {attack: 0.001, decay: 0.3, release: 0.8}`, `harmonicity: 3.1`, `modulationIndex: 12`. Wet 30% through `Reverb` decay 0.9s. Pitch-shifted down 4 semitones from default.
- Character: A dense organic thud with a brief metallic ring — something dense dropped on stone.

**3. Bait scent active — ambient pulse**
- Trigger: Looping while bait is on floor and beast has not yet reached it.
- Tone.js: `Synth` with `oscillator.type: "sine"`, frequency `55Hz`, very low amplitude (`0.04`), `LFO` on volume at `0.5Hz`, swing between `0.02` and `0.06`. Barely audible — subliminal.
- Character: The bait is breathing. The player should feel it more than hear it.

**4. Beast eating — wet chewing loop**
- Trigger: Beast reaches bait and enters EATING state. Loops for full eat duration.
- Tone.js: `NoiseSynth` with `noise.type: "brown"`, low-pass filter at `200Hz`, envelope sustain at `0.3`. Irregular amplitude via `LFO` at `1.3Hz`, `type: "random"`. Add `Distortion` at `0.2` wet.
- Character: Muffled, wet, irregular. Do not make it cute. It should be unsettling.

**5. Beast alert — head snap**
- Trigger: The instant the beast's attention cone pivots toward the player (not bait — toward PLAYER specifically).
- Tone.js: Single `Synth` hit with `oscillator.type: "sawtooth"`, pitch sweep from `C3` (130Hz) to `G2` (98Hz) over `0.15s`, envelope `{attack: 0.005, decay: 0.15, sustain: 0, release: 0}`. No reverb. Dry.
- Character: A sharp, dry downward scrape — like something large has just noticed something small.

**6. Footstep ripple — stone (Level 3+ only)**
- Trigger: Each player step on stone floor tile.
- Tone.js: `Synth` with `oscillator.type: "triangle"`, frequency `440Hz`, envelope `{attack: 0.001, decay: 0.08, sustain: 0, release: 0}`, amplitude `0.05`. Apply `Reverb` `decay: 2.4s`, `wet: 0.6`. The reverb tail is the point — it lingers in the stone.
- Character: A tiny, cold, precise tap. On stone. The reverb does the terror work — it says *this floor tells everything.*

---

## MECHANIC SPEC

### Core Loop
```
SCAN → PREDICT → DROP → RUN → SCAN
```
The player observes the beast's current head direction and sweep arc, predicts where the cone will be in 2–4 seconds, drops bait in the optimal zone to redirect attention, and moves through the cleared corridor. Then immediately scans again — the beast will finish eating in a fixed duration.

### Primary Input
- **Keyboard:** `SPACE` or `DOWN ARROW` to drop bait. Left/right arrow to move. No jump. No crouch toggle — the player is always crouched (posture is permanent).
- **Mouse/Touch fallback:** Click/tap floor tile in front of player to drop bait there (bait drops at player's feet, not at click point — touch is just an alternate drop trigger, not a targeting system).
- **Stop moving:** No key held = player stops instantly. This is intentional — freeze is a valid action. Do not add momentum or slide.

### Bait System

**Drop mechanics:**
- Bait is dropped at player's current tile.
- Bait cannot be picked up once dropped.
- One bait active at a time — dropping a second bait while one is active cancels the first (it disappears with a brief dim flash, no scent emitted). This is advanced play: replacing a poorly-placed bait.

**Scent radius:**
- Bait emits scent in a circle of radius `R` tiles.
- If beast is within `R` tiles AND beast's attention is not already locked on another target, beast pivots toward bait and walks to it.
- `R` varies per level (see Level Design).
- **There is no visual circle showing `R`.** The only hint is the beast's ear animation (see below).

**Beast ear animation as scent hint:**
- The beast has two small ear-fin shapes on its silhouette head.
- When bait is dropped within the current `R` distance, the ears rotate forward (toward bait) in 0.3 seconds BEFORE the head pivots. This is the tell.
- When bait is dropped outside `R`, ears do not animate. The beast does not detect it.
- The ear animation is the game's only "scent radius indicator." Players must learn to read it.
- This is the Gran Turismo corner: you don't see the safe zone, you feel where it is after 10 attempts.

**Eat duration:** How long the beast stays at the bait eating before resuming sweep.

| Level | Eat duration |
|---|---|
| 1 | 6.0 seconds |
| 2 | 4.5 seconds |
| 3 | 3.5 seconds |
| 4 | 3.0 seconds |
| 5 | 2.5 seconds |

**Detection — player proximity:**
- If the player is within `D` tiles of the beast AND within the beast's current attention cone (90° arc in front of beast's head), the beast locks on the player. This triggers CHASE state.
- `D` = 2.5 tiles in all levels. Does not change. (The danger zone is fixed — what changes is how much time you have.)

**CHASE state:**
- Beast moves toward player at 1.8x normal walk speed (faster than player run speed — player cannot outrun the beast).
- Beast ignores bait completely during CHASE.
- Chase is broken ONLY by: player reaching exit door (stage clear) OR player moving off-screen / out of line-of-sight around a corner for 4.0 continuous seconds. If LoS is broken for 4s, beast returns to sweep.
- **Bait does not break CHASE.** This is deliberate — dropping bait in panic when you've already been spotted does nothing. The only escape is the corner.

### Dead State

**How you die:**  
Beast reaches player tile during CHASE state. No animation — the screen cuts to black in 1 frame.

**Death feedback:**  
Black screen. 0.5 seconds silence. Then: single low 80Hz `MembraneSynth` thud. Then: the level fades back in exactly as it was at level start — player at start position, bait count reset to starting count, beast at its starting position and heading. No death screen. No counter. No "YOU DIED." You are just back. The memory of what went wrong is all you have.

**Why no death animation:**  
If you see the beast reach you, you already know what happened. The instant cut to black IS the horror. It says: when that thing reaches you, it is over before you can register it.

**Respawn:**  
Immediate. Level resets in full. 0.8 second fade-in from black. No delay beyond that.

### Player Movement Speed
- **Walk speed:** 2.0 tiles/second (default, when no key held is 0)
- **Run speed:** 3.5 tiles/second (player always runs when moving — this is not a toggle, this is just the one speed)
- Actually: the player has only one speed when moving. Call it "sprint." 3.5 tiles/second. Stopping is instant.

**Footstep ripples (Level 3+):**
- Every player step on a STONE floor tile emits a ripple ring.
- Ring expands from player feet: starts at 4px radius, expands to 48px radius over 1.2 seconds, then disappears.
- Ring color: `#A0A0A0` at 10% opacity.
- The small creature (introduced Level 3) detects footstep ripples within 3 tiles. If a ripple spawns within 3 tiles of the small creature's position, it moves toward that origin point.
- The small creature checks ripple origin, not player position — if the player has moved away, the creature walks to where the sound was, not where the player is now.
- **SOFT FLOOR tiles:** Certain tile types (introduced Level 3) produce no ripple. Small creature cannot detect movement on these. Soft floor is visually distinct: slightly warmer tone (`#221D14` vs stone `#1C1812`), slightly textured (4px noise pattern). Players discover this by stepping on it and watching the small creature fail to react.

### Beast Sweep Pattern
- Beast has a home sweep: rotates head left-right through a fixed arc (set per level, see Level Design).
- Head completes one full sweep (left extent to right extent back to left) in a fixed period (sweep period, set per level).
- During sweep, beast walks a short patrol route (2–4 tiles, set per level) — it is not stationary.
- When detecting bait: beast pivots head toward bait, walks to bait tile, eats for eat duration, then resumes sweep from bait tile. The beast does NOT return to patrol start after eating — it resumes from wherever it is. **This is important.** A bait dropped in the wrong place extends the creature's patrol zone permanently for that run.

### Score System
There is no score. There is no timer visible to the player. There are no stars. There are no coins.

**What the player earns:** Reaching the exit door. That's it. The reward is the door opening.

**Internal tracking (not displayed to player):**
- Bait remaining when exit reached (for developer telemetry only — not shown)
- Deaths per level (for balancing only — not shown)

If a score or progress indicator is ever added to this game in future development, it should be discussed as a separate design decision. The current spec explicitly excludes any scoring UI.

### Bait Count Display
The player's bait count is shown as physical items in a strip at the top-left of the screen: small amber dots, not numbers. Maximum 5 dots (maximum starting bait is 5).

- Each dot: 6px circle, `#C8A96E` fill, slight bloom at `0.3` strength
- Dot disappears (fades over 0.3s) when bait is dropped
- No "0 bait" text — when all dots are gone, the strip is empty and the player's hand animation changes: arm hangs lower, no longer extended forward

This is the only HUD element. No health bar. No timer. No minimap.

---

## LEVEL DESIGN

### Level 1 — *THE LESSON*
**What's new:** Everything. This is tutorial-through-play.  
**Duration:** 30–90 seconds (varies by player skill — short for skilled players, survivable for new ones)  
**Layout:** Single corridor, left to right. No junctions. One beast.  
**Beast parameters:**
- Sweep arc: ±40° from forward
- Sweep period: 5.0 seconds (slow, readable)
- Patrol route: 3-tile back-and-forth, centered on corridor midpoint
- Walk speed: 1.2 tiles/second
**Bait count:** 3
**Scent radius:** 4.0 tiles (generous — bait works from well away)  
**Stone floor:** All stone. No soft floor. No ripples. (Soft floor and ripple mechanics don't exist yet — introduced in Level 3.)  
**Eat duration:** 6.0 seconds  
**No second creature.**  
**The task:** Drop one bait on the near side of the beast, wait for it to turn, walk past, reach the door. The player can solve this level with 1 bait if they understand the mechanic. Most first-time players will use 2.

**Design note:** The corridor is wide enough that the player can see the bait glow, the beast, and the exit door all at the same time in the initial viewport. The entire game's logic is readable in the opening frame.

---

### Level 2 — *THE JUNCTION*
**What's new:** A branching path. Two routes to the exit. One route is shorter but requires threading closer to the beast.  
**Duration:** 60–180 seconds  
**Layout:** Main corridor with a Y-junction at the midpoint. Upper path is longer but the beast doesn't patrol there. Lower path is direct but the beast's patrol overlaps it for 2 of its 5-tile patrol route.  
**Beast parameters:**
- Sweep arc: ±50° from forward
- Sweep period: 4.0 seconds
- Patrol route: 5-tile route along the lower fork approach
- Walk speed: 1.4 tiles/second  
**Bait count:** 4  
**Scent radius:** 3.5 tiles  
**Eat duration:** 4.5 seconds  
**No second creature.**  
**The task:** Player must either route around (safe, uses no bait, but reveals that bait is a resource to be conserved) or lure the beast off the lower path (direct, uses 1 bait, saves time). The choice teaches bait conservation. Skilled players take the lower route and save all 4 bait.

**New mechanic introduced:** After the beast finishes eating and resumes patrol, its route has shifted — it now patrols from the bait drop point, not its original start. This means a carelessly placed bait in Level 2 makes the return route more dangerous. The player will learn this the hard way.

---

### Level 3 — *THE LISTENER*
**What's new:** Second creature. Footstep ripples. Soft floor tiles.  
**Duration:** 90–240 seconds  
**Layout:** Main corridor with a central stone-floor chamber and a soft-floor side passage. Exit requires passing through the chamber.  
**Beast 1 (the Lurer):**
- Sweep arc: ±45°
- Sweep period: 3.8 seconds
- Patrol route: 4-tile route blocking chamber entrance
- Walk speed: 1.5 tiles/second  
**Beast 2 (the Listener):**
- Size: 60% of Beast 1 (56px tall vs 84px)
- Does NOT respond to bait. Ear animation is absent — it has no ears. Visual reminder that this one hears, not smells.
- Patrol route: 3-tile route in the chamber itself
- Walk speed: 1.8 tiles/second (faster — it is more agile, smaller)
- Detection radius for footstep ripples: 3.0 tiles
- If it detects a ripple, it walks to the ripple origin. If the player is still there when it arrives: CHASE.
- Its attention cone for player visual detection: ±30° (narrower than Beast 1) — it navigates more by sound than sight.
**Bait count:** 4  
**Scent radius:** 3.0 tiles  
**Eat duration:** 3.5 seconds  

**The WOW MOMENT lives here.**  
The player, for the first time, must be still. Not because a timer says so. Not because a UI prompt says so. But because they see the small creature moving toward where their footstep rings appeared, and they understand the system in their gut. The player freezes. The first beast is chewing bait 3 tiles left. The small creature circles toward the last ripple origin. The player does not breathe. This is the best moment in the game, and the programmer did not have to write a single line of tutorial text to create it.

**Soft floor placement:** The soft-floor side passage is 4 tiles wide and curves around the chamber, rejoining the main corridor behind Beast 2's patrol. The player can route around Beast 2 entirely by using the soft floor — but this route requires passing closer to Beast 1, so bait is still needed.

---

### Level 4 — *THE LOOP*
**What's new:** Introduces the bait loop possibility. The level is architecturally designed so that an expert player can engineer a bait configuration the beast cycles between.  
**Duration:** 120–300 seconds  
**Layout:** Cruciform — a central hub with four corridors. Exit is in the north corridor. Start is in the south. Beast 1 patrols the east-west cross corridor. Beast 2 (the Listener) patrols the north corridor near the exit.  
**Beast 1:**
- Sweep arc: ±60°
- Sweep period: 3.2 seconds
- Patrol: full east-west corridor, 8 tiles
- Walk speed: 1.6 tiles/second  
**Beast 2 (the Listener):**
- Patrol: 5-tile patrol in north corridor
- Walk speed: 2.0 tiles/second  
**Bait count:** 5  
**Scent radius:** 2.8 tiles  
**Eat duration:** 3.0 seconds  

**The bait loop exploit (intentional, reward for expert play):**  
If the player drops Bait A in the east alcove and Bait B in the west alcove of the cross corridor, Beast 1 will detect A, walk east, eat (3.0s), then resume sweep from east position, detect B in west alcove, walk west, eat, detect A again (if timing is right — bait lasts 8 seconds on the floor before its scent fades). With precise placement, Beast 1 can cycle between the two drops indefinitely, never returning to its north-south blocking position.

**Bait scent fade:** Bait scent fades after 8 seconds on the floor. The bait item remains visible but stops emitting scent (amber pulse dims to `#6B4F1A` — the dim resting color). A new mechanic introduced here: the player must manage bait timing, not just position.

**Visual indicator of bait fade:** The bait amber pulse slows as it approaches fade — pulse rate drops from 1.2Hz to 0.3Hz over the last 3 seconds, then stops. This is the only warning. No text. No timer.

---

### Level 5 — *THE THREADING*
**What's new:** Nothing mechanical. The climax uses only what the player knows. This level is a test, not a tutorial.  
**Duration:** 90–240 seconds  
**Layout:** Three narrow corridors in sequence, connected by small transition cells. No Y-junctions — only one path. Beast 1 patrols corridor 1. Both beasts occupy corridor 2. Beast 2 (the Listener) patrols corridor 3. Corridor 3 is entirely stone floor. The exit is at the end of corridor 3.  
**Beast 1:**
- Sweep arc: ±55°
- Sweep period: 2.8 seconds
- Walk speed: 1.7 tiles/second  
**Beast 2 (corridor 2):**
- Sweep arc: ±45°
- Sweep period: 3.0 seconds
- Walk speed: 1.6 tiles/second  
**Beast 3 (the Listener, corridor 3):**
- Walk speed: 2.0 tiles/second  
**Bait count:** 3 (tight — the level is designed to require exactly 2 bait used optimally, or 3 used acceptably)  
**Scent radius:** 2.5 tiles (smallest in game — the window is very tight)  
**Eat duration:** 2.5 seconds  

**The designed climax sequence:**  
Corridor 2 has a 2-tile-wide soft-floor strip running along the south wall. The player must lure Beast 2 (in corridor 2) north using one bait, then sprint the soft-floor strip while Beast 3 (the Listener) is at the far end of its patrol. Corridor 3 is all stone — the player must freeze mid-corridor while Beast 3 completes its return sweep, then sprint the final 4 tiles to the door.

**One bait must be held in reserve for corridor 3** — if Beast 3 detects footsteps and pivots toward player, the bait can be dropped behind the player to lure it away while the player runs for the door. The player who saves this bait wins. The player who used it earlier does not.

**Exit door behavior:** The door is always visible, always glowing. When the player enters the exit tile, the door swings open (0.4 second animation), light floods the viewport from the right edge, all beast audio cuts to silence, and after 1.2 seconds of just the door-light sound (a warm, low resonance — single `Synth` note `A3`, slow attack 0.8s, held 1.0s, decay 1.5s), the level ends. No score screen. Cut to black. The next level fades in.

---

## THE MOMENT

Standing in the stone chamber in Level 3, bait exhausted, the small creature's head tracking toward the fading ring where your last footstep landed — and you realize you have not moved for eight seconds, your finger is not touching a key, and the only thing protecting you is the quality of your stillness.

---

## EMOTIONAL ARC

**First 30 seconds:**  
The massive creature is already there when the level loads. It is not a reveal — it is the first thing you see. The glowing item in your hand is the second. You feel the imbalance immediately: the creature fills half the corridor and you are a 28-pixel smudge. You drop the bait because there is nothing else to do. The creature's head snaps left. The path clears. You run. In those four seconds you understand: this game is about controlling something enormous with something tiny. You feel powerful and terrified simultaneously.

**After 2 minutes (the groove):**  
You have stopped thinking about the bait and started thinking about the beast. Its sweep arc is a clock. You can feel when it's about to rotate back — you've calibrated to its rhythm. You drop bait with a half-second of margin before the cone would reach you, and you're already moving before the head turns. This fluency feels like cheating. It is not cheating. It is mastery. The dread doesn't go away — it compresses. You're now operating closer to the margins by choice, and you know it.

**Near the win (climax):**  
Corridor 3. Stone floor. Last bait in hand. Beast 3 at far end of sweep, starting to turn back. You know the math: 2.5 tile sweep at 2.0 tiles/second means 1.25 seconds until its cone reaches you. You are 4 tiles from the door. At 3.5 tiles/second you will make it in 1.14 seconds. But you froze to let it pass. You've been still for 3 seconds. Your ripple rings have faded. You stand up and run. You make it with 0.1 seconds to spare, or you don't. Either way, you knew the exact number. That's this game.

---

## THIS GAME'S IDENTITY IN ONE LINE

**"This is the game where you see exactly where the monster is, know precisely how long you have, and are still wrong."**

---

## START SCREEN

### Layout
- Background: `#0D0B08` full fill
- Vignette: `0.80` strength (heavier than gameplay — the start screen is a held breath)
- No UI text except the game title and one instruction: a single amber dot that pulses, positioned below the title, that disappears when the player presses any key. No "PRESS SPACE TO START." Just the dot. The player presses something. It disappears. The game starts.

### Idle Animation (game-world specific — NOT generic particles)
**Bait amber drift with beast sweep:**  
In the background of the start screen, behind the title, a very large beast silhouette (160px tall, filling the right half of the screen) slowly sweeps its head left and right — the same ear-forward animation and cone it uses in gameplay, but rendered at 15% opacity so it reads as atmospheric rather than threatening. 

The beast completes one slow sweep every 8 seconds.

Every 5–7 seconds (random), a small amber dot — a bait — materialises at a random position in the lower 40% of the screen, pulses three times (1.2Hz, growing bloom radius 8px → 24px), then fades over 1.5 seconds. The beast's head pivots toward each bait as it appears, tracking it. The beast never moves toward them — just the slow head tracking. A demonstration of the core mechanic, rendered as ambiance.

This establishes the world, the aesthetic, and the core interaction (creature tracks bait) without a single word.

### SVG Option A (required) — Glow Title

```svg
<svg viewBox="0 0 600 120" xmlns="http://www.w3.org/2000/svg">
  <defs>
    <filter id="amber-glow" x="-30%" y="-80%" width="160%" height="360%">
      <!-- Step 1: blur the source text for glow halo -->
      <feGaussianBlur in="SourceGraphic" stdDeviation="6" result="blur-wide"/>
      <!-- Step 2: tighter inner glow -->
      <feGaussianBlur in="SourceGraphic" stdDeviation="2" result="blur-tight"/>
      <!-- Step 3: composite tight glow over wide halo, both over original -->
      <feComposite in="blur-tight" in2="blur-wide" operator="over" result="combined-blur"/>
      <feComposite in="SourceGraphic" in2="combined-blur" operator="over" result="glowed"/>
      <!-- Step 4: colorize the glow amber -->
      <feColorMatrix in="glowed" type="matrix"
        values="1.4  0.3  0    0  0.05
                0.8  1.1  0    0  0
                0    0    0.2  0  0
                0    0    0    1  0" result="amber-tint"/>
    </filter>
    <!-- Separate filter for ultra-bright core -->
    <filter id="core-glow" x="-10%" y="-50%" width="120%" height="200%">
      <feGaussianBlur in="SourceGraphic" stdDeviation="1" result="core-blur"/>
      <feComposite in="SourceGraphic" in2="core-blur" operator="over"/>
    </filter>
  </defs>
  <!-- Wide amber halo layer -->
  <text x="300" y="88" 
    font-family="'Courier New', monospace" 
    font-size="64" 
    font-weight="700"
    letter-spacing="12"
    text-anchor="middle" 
    fill="#C8A96E"
    filter="url(#amber-glow)"
    opacity="0.7">SCENT TRAIL</text>
  <!-- Core bright layer -->
  <text x="300" y="88" 
    font-family="'Courier New', monospace" 
    font-size="64" 
    font-weight="700"
    letter-spacing="12"
    text-anchor="middle" 
    fill="#F0D49A"
    filter="url(#core-glow)">SCENT TRAIL</text>
</svg>
```

**Animation:** The entire SVG group slowly pulses between opacity `0.85` and `1.0` over 2.4 seconds, eased `sin`. Same rhythm as the bait in gameplay. The title is breathing.

### SVG Option B — Beast Silhouette

```svg
<svg viewBox="0 0 400 200" xmlns="http://www.w3.org/2000/svg">
  <!-- Beast silhouette: massive hunched form, head lowered, facing left -->
  <!-- Built from paths, not images. Pure vector. -->
  <defs>
    <radialGradient id="eye-shine" cx="50%" cy="50%" r="50%">
      <stop offset="0%" stop-color="#C42222" stop-opacity="1"/>
      <stop offset="60%" stop-color="#8B1A1A" stop-opacity="0.8"/>
      <stop offset="100%" stop-color="#8B1A1A" stop-opacity="0"/>
    </radialGradient>
  </defs>
  <!-- Beast body — fill slightly darker than background -->
  <path d="M 60 160 
           C 60 160, 40 140, 45 110 
           C 50 80, 80 60, 120 55 
           C 160 50, 200 52, 240 48 
           C 280 44, 320 30, 350 20 
           C 380 10, 390 18, 385 35 
           C 380 55, 340 65, 300 72 
           C 260 78, 220 82, 200 90 
           C 180 98, 175 115, 180 135 
           C 185 155, 175 170, 160 175 
           C 145 180, 120 178, 100 175 
           C 80 172, 65 168, 60 160 Z" 
        fill="#0A0806"/>
  <!-- Neck and head extending left -->
  <path d="M 80 100 
           C 70 90, 50 85, 30 82 
           C 10 79, 5 88, 8 98 
           C 11 108, 25 112, 45 110 
           C 65 108, 78 108, 80 100 Z"
        fill="#0A0806"/>
  <!-- Head mass -->
  <path d="M 8 82 
           C -5 75, -12 80, -10 92 
           C -8 104, 5 112, 20 108 
           C 35 104, 40 95, 35 88 
           C 30 81, 18 78, 8 82 Z"
        fill="#0A0806"/>
  <!-- Eye shine — right eye -->
  <ellipse cx="10" cy="90" rx="4" ry="3" fill="url(#eye-shine)"/>
  <!-- Ear fin — forward position (alert) -->
  <path d="M 20 80 C 15 70, 22 65, 28 72 C 34 79, 28 83, 20 80 Z" fill="#0A0806"/>
  <!-- Legs — four, partial, heavy -->
  <rect x="85" y="155" width="18" height="30" rx="3" fill="#0A0806"/>
  <rect x="120" y="158" width="16" height="27" rx="3" fill="#0A0806"/>
  <rect x="155" y="156" width="18" height="29" rx="3" fill="#0A0806"/>
  <rect x="188" y="154" width="17" height="31" rx="3" fill="#0A0806"/>
</svg>
```

**Usage:** Positioned in the lower-right of start screen at 40% opacity. Partially occluded by the right vignette. Only the eye-shine and the bulk of the torso are clearly visible. The player sees the scale — and their own 28px silhouette should be placed in the lower-left for contrast.

---

## PROGRAMMER IMPLEMENTATION NOTES

These are not optional suggestions. These are constraints.

**1. Beast movement is smooth, not tile-snapping.**  
The beast walks at a continuous speed to a target tile. It does not teleport between tiles. Path is interpolated. This matters because the player must read the beast's travel time visually.

**2. The attention cone is always rendered.**  
The cone is a translucent wedge in front of the beast's head. Color: `#8B1A1A` at 8% opacity normally, `#C42222` at 18% opacity when ALERTED. It is always visible. This is what Metal Slug does with its bullet arcs. The player is never surprised by where the cone is.

**3. Player movement is instant-stop, no deceleration.**  
No slide. No easing on stop. The player stops the frame the key is released. This is critical — the dread of a near-miss requires precision. Slide would make the game feel unpredictable.

**4. Bait is dropped at player tile, not aimed.**  
One button. No targeting. No mouse-click-to-drop. The depth of the game comes from positioning your body correctly before dropping, not from pointing and clicking. This keeps the tension physical.

**5. Stone floor type is a tile property, not a visual guess.**  
Each tile has a `floorType` property: `stone` or `soft`. Ripple emission is `if (floorType === 'stone')`. Visual distinction follows the tile type map, not a separate asset.

**6. Beast 2 (the Listener) has no scent detection.**  
No bait-detection code runs for Beast 2 at all. If bait is dropped near it, nothing happens. This behavioral difference must be visually readable via the absent ear animation. The creature's silhouette has no ear fins. This is the tell.

**7. The "bait loop" is emergent, not scripted.**  
Do not script the bait loop scenario. The level architecture (two alcoves within scent radius of Beast 1's patrol) creates the emergent possibility. The beast's post-eat resume behavior (patrolling from new position, not returning to start) enables it. Both of these are already in the spec. The loop is a consequence, not a feature.

**8. No score. No time. No text.**  
The only UI elements are: bait count (amber dots, top-left), attention cone (always on beast), ripple rings (on stone floor steps, Level 3+), and bait scent rings (small expanding ring from drop point, fades over 0.8s). No other UI elements exist in this version.

---

*Scent Trail — Round 13 Design Document*  
*Every ambiguity removed. Every frame argued over.*

```
END_DESIGN_DOC
```
