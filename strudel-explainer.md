# Strudel Codebase Documentation

## Table of Contents

1. [Project Overview](#project-overview)
2. [Architecture Overview](#architecture-overview)
3. [Core Concepts](#core-concepts)
4. [Package Ecosystem](#package-ecosystem)
5. [Pattern Engine Deep Dive](#pattern-engine-deep-dive)
6. [Mini Notation Parser](#mini-notation-parser)
7. [Transpiler System](#transpiler-system)
8. [Audio Synthesis (SuperDough)](#audio-synthesis-superdough)
9. [Website and REPL](#website-and-repl)
10. [How Everything Connects](#how-everything-connects)
11. [Key Files Reference](#key-files-reference)
12. [Code Examples](#code-examples)

---

## Project Overview

**Strudel** is a browser-based live coding music environment that brings the power of [TidalCycles](https://tidalcycles.org/) to JavaScript. It allows users to create complex musical patterns using a combination of mini notation (a concise DSL for rhythmic patterns) and JavaScript code, all running entirely in the browser.

**Key Features:**
- **Pattern-based composition**: Create musical patterns using time-based transformations
- **Mini notation**: Concise syntax for expressing rhythmic patterns (e.g., `"bd sd hh sd"`)
- **JavaScript integration**: Combine patterns with the full power of JavaScript
- **Web Audio**: Built-in audio synthesis and sampling using Web Audio API
- **Live coding friendly**: Designed for real-time performance and experimentation
- **Multi-output**: Support for MIDI, OSC, Hydra visuals, and more

**Project Structure:**
- **Monorepo**: Uses pnpm workspaces with 27+ packages
- **License**: GNU Affero General Public License v3 (AGPL-3.0-or-later)
- **Primary Website**: https://strudel.cc
- **Build System**: Vite for building, Astro for the website

---

## Architecture Overview

Strudel follows a layered architecture:

```
┌─────────────────────────────────────────────────┐
│          Website / REPL Interface               │
│         (Astro + React + CodeMirror)            │
└─────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────┐
│            Transpiler Layer                     │
│    (Converts user code to executable patterns)  │
└─────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────┐
│         Pattern Generation Layer                │
│  Core + Mini Notation + Tonal + Controls        │
└─────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────┐
│          Output / Scheduling Layer              │
│     (Clock + Scheduler + Event Handling)        │
└─────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────┐
│              Output Targets                     │
│  WebAudio | MIDI | OSC | Hydra | Draw | etc.   │
└─────────────────────────────────────────────────┘
```

**Data Flow:**
1. User writes code in the REPL
2. Transpiler transforms code (adds location tracking, handles mini notation strings)
3. Code evaluates to create Pattern objects
4. Clock queries patterns for events in time windows
5. Events are scheduled and sent to output targets (audio, visuals, etc.)

---

## Core Concepts

### 1. **Hap (Event)**

A `Hap` represents a single event in time with a value. It's the fundamental unit of patterns.

**Key Properties:**
- `whole`: The complete timespan of the event (TimeSpan)
- `part`: The fragment of the event being queried (TimeSpan)
- `value`: The actual data (could be a string, number, or object)
- `context`: Metadata like source code locations
- `stateful`: Whether the value is a stateful function

**File**: `/home/user/strudel/packages/core/hap.mjs`

```javascript
class Hap {
  constructor(whole, part, value, context = {}, stateful = false) {
    this.whole = whole;
    this.part = part;
    this.value = value;
    this.context = context;
    this.stateful = stateful;
  }
}
```

### 2. **TimeSpan**

Represents a span of time from `begin` to `end` using rational fractions.

**Key Properties:**
- `begin`: Start time (Fraction)
- `end`: End time (Fraction)
- `duration`: Calculated as `end - begin`

**File**: `/home/user/strudel/packages/core/timespan.mjs`

```javascript
class TimeSpan {
  constructor(begin, end) {
    this.begin = Fraction(begin);
    this.end = Fraction(end);
  }

  get duration() {
    return this.end.sub(this.begin);
  }
}
```

### 3. **Pattern**

The core abstraction in Strudel. A Pattern is a function that takes a `State` (containing a time window) and returns an array of `Hap` events.

**Key Properties:**
- `query`: Function that maps State → Array<Hap>
- `_steps`: Number of steps per cycle (for visualization)

**File**: `/home/user/strudel/packages/core/pattern.mjs`

```javascript
class Pattern {
  constructor(query, steps = undefined) {
    this.query = query;
    this._steps = steps;
  }
}
```

### 4. **State**

Encapsulates the query context passed to patterns.

**Properties:**
- `span`: The TimeSpan being queried
- `controls`: Control values (for parameter passing)

**File**: `/home/user/strudel/packages/core/state.mjs`

### 5. **Fraction**

Uses the `fraction.js` library to represent time using rational numbers, ensuring perfect precision for rhythmic calculations.

**File**: `/home/user/strudel/packages/core/fraction.mjs`

---

## Package Ecosystem

### Core Packages

#### **@strudel/core** (v1.2.2)
The foundation of Strudel containing the pattern engine.

**Key Files:**
- `pattern.mjs`: Pattern class and combinators (3000+ lines)
- `hap.mjs`: Event representation
- `timespan.mjs`: Time interval representation
- `controls.mjs`: Audio control parameters (1700+ lines)
- `zyklus.mjs`: Clock implementation
- `euclid.mjs`: Euclidean rhythm generation
- `signal.mjs`: Continuous signal patterns
- `util.mjs`: Utility functions

**Key Functions:**
- Pattern combinators: `sequence`, `stack`, `fastcat`, `slowcat`, `cat`
- Transformations: `fast`, `slow`, `rev`, `every`, `jux`
- Temporal operations: `early`, `late`, `off`, `iter`
- Control methods: `.note()`, `.s()`, `.gain()`, `.pan()`, etc.

**Dependencies:**
- `fraction.js`: Rational number arithmetic

#### **@strudel/mini** (v1.2.2)
Mini notation parser that converts compact string syntax into patterns.

**Key Files:**
- `krill.pegjs`: PEG grammar definition (600+ lines)
- `krill-parser.js`: Generated parser from pegjs (3300+ lines)
- `mini.mjs`: AST to Pattern converter (300+ lines)

**Features:**
- Sequence parsing: `"bd sd hh sd"`
- Polyrhythm: `"bd [sd cp] hh"`
- Speed multiplication: `"bd*2 sd"`
- Rests: `"bd ~ hh ~"`
- Euclidean rhythms: `"bd(3,8)"`
- Operators: `!` (replicate), `/` (slow), `*` (fast), `@` (stretch)

**Parser Generation:**
```bash
npm run build:parser  # Generates krill-parser.js from krill.pegjs
```

#### **@strudel/transpiler** (v1.2.2)
Transforms user code into evaluatable Strudel code.

**Key Features:**
- Parses JavaScript with Acorn
- Transforms AST with estree-walker
- Generates code with escodegen
- Auto-wraps quoted strings with mini notation
- Adds location tracking for highlighting
- Handles template literals for Tidal code
- Widget detection for UI controls

**File**: `/home/user/strudel/packages/transpiler/transpiler.mjs`

**Transformations:**
```javascript
// Input
"bd sd hh sd"

// After transpilation
mini("bd sd hh sd").withLocation(...)
```

**Dependencies:**
- `acorn`: JavaScript parser
- `escodegen`: Code generator
- `estree-walker`: AST traversal

#### **@strudel/tonal** (v1.2.2)
Musical theory integration for notes, scales, and chords.

**Features:**
- Note name to frequency conversion
- Scale generation
- Chord voicings
- Interval calculations

**Dependencies:**
- `@tonaljs/tonal`: Music theory library
- `chord-voicings`: Chord voicing generation
- `webmidi`: MIDI note utilities

#### **@strudel/webaudio** (v1.2.3)
Web Audio output integration, wraps superdough.

**Key Files:**
- `webaudio.mjs`: Pattern scheduler for Web Audio
- `scope.mjs`: Oscilloscope visualization
- `spectrum.mjs`: Frequency spectrum analysis

**Exports**: Re-exports everything from `superdough`

### Audio & Synthesis

#### **superdough** (v1.2.3)
Web Audio sampler and synthesizer (standalone package).

**Key Files:**
- `superdough.mjs`: Main audio engine (600+ lines)
- `sampler.mjs`: Sample loading and playback (350+ lines)
- `synth.mjs`: Synthesizer implementation (300+ lines)
- `worklets.mjs`: Audio Worklet processors (700+ lines)
- `helpers.mjs`: Web Audio utility functions
- `reverb.mjs`: Reverb effect
- `feedbackdelay.mjs`: Delay effect
- `vowel.mjs`: Vowel filter effect
- `zzfx.mjs`: Sound effect synthesizer

**Key Features:**
- Sample playback with streaming
- Waveform synthesis (sawtooth, square, triangle, sine)
- Effects: delay, reverb, filters, distortion, phaser, vowel
- Multi-channel orbit system for effect buses
- Polyphony management (default 128 voices)
- Sound registration system
- ADSR envelope

**Audio Parameters:**
- Sound selection: `s`, `n`, `bank`
- Amplitude: `gain`, `velocity`
- Filters: `cutoff`, `resonance`, `hcutoff`, `hresonance`, `bandf`, `bandq`
- Effects: `delay`, `delaytime`, `delayfeedback`, `room`, `size`
- Distortion: `crush`, `distort`
- Spatial: `pan`, `orbit`
- Pitch: `note`, `freq`, `speed`
- Envelope: `attack`, `decay`, `sustain`, `release`
- Sample control: `begin`, `end`, `clip`

**Orbit System:**
Each pattern can output to different "orbits" (effect buses), allowing independent effect processing for different patterns.

### UI & Editor

#### **@strudel/codemirror** (v1.2.2)
CodeMirror 6 extensions for Strudel.

**Features:**
- Syntax highlighting
- Auto-completion for Strudel functions
- Multiple keymap support (default, vim, emacs, VSCode)
- Flash highlighting for active code
- Mini notation syntax support

**Dependencies:**
- CodeMirror 6 packages
- `@replit/codemirror-vim`, `@replit/codemirror-emacs`: Keymaps

#### **@strudel/repl** (v1.2.2)
REPL component logic (used by website).

**Key Files:**
- `prebake.mjs`: Pre-bakes patterns for instant playback
- `repl-component.mjs`: Reusable REPL component

### Output Integrations

#### **@strudel/midi** (v1.2.3)
MIDI output support.

**Features:**
- Send patterns to MIDI devices
- Note on/off messages
- CC (Control Change) messages

**Dependencies:**
- `webmidi`: Web MIDI API wrapper

#### **@strudel/osc** (v1.2.2)
OSC (Open Sound Control) messaging.

**Features:**
- Send patterns via OSC
- Compatible with SuperCollider, Max/MSP, etc.
- Includes OSC server implementation

**Dependencies:**
- `osc-js`: OSC protocol implementation

#### **@strudel/hydra** (v1.2.2)
Integration with Hydra video synthesizer.

**Features:**
- Send pattern data to Hydra
- Create audio-reactive visuals
- Sync patterns with visuals

**Dependencies:**
- `hydra-synth`: Hydra video synth engine

#### **@strudel/draw** (v1.2.2)
Pattern visualization and drawing helpers.

**Features:**
- Pianoroll visualization
- Circle pattern visualization
- Canvas drawing utilities

#### **@strudel/csound**
Csound integration for advanced synthesis.

#### **@strudel/serial**
Serial port communication for hardware control.

#### **@strudel/mqtt**
MQTT messaging for IoT integration.

#### **@strudel/gamepad**
Gamepad/controller input support.

#### **@strudel/motion**
Motion sensor input (accelerometer, gyroscope).

### Special Packages

#### **@strudel/soundfonts**
SoundFont (.sf2) file support for realistic instrument sounds.

#### **@strudel/sampler**
Dedicated sample server implementation.

#### **@strudel/tidal** (v0.2.0)
Experimental Haskell Tidal code interpreter.

**Dependencies:**
- `hs2js`: Haskell to JavaScript transpiler

#### **@strudel/hs2js**
Haskell to JavaScript transpilation using tree-sitter.

**Features:**
- Parse Haskell Tidal code
- Convert to JavaScript Strudel patterns

#### **@strudel/xen**
Xenharmonic (microtonal) music support.

**Features:**
- Custom tuning systems
- Microtonal scales

#### **@strudel/desktopbridge**
Bridge for Tauri desktop app integration.

#### **@strudel/embed**
Embeddable Strudel player for websites.

#### **@strudel/reference**
API reference documentation generator.

#### **@strudel/web**
Web-specific utilities and helpers.

#### **vite-plugin-bundle-audioworklet**
Vite plugin for bundling Audio Worklets.

---

## Pattern Engine Deep Dive

### How Patterns Work

Patterns are **lazy** and **infinite**. They don't generate events until queried for a specific time window.

**Query Process:**

1. A `State` object is created with a `TimeSpan` (e.g., 0 to 1)
2. The pattern's `query` function is called with this State
3. The query function returns an array of `Hap` events
4. Each Hap contains timing and value information

```javascript
// Create a simple pattern
const pattern = sequence('a', 'b', 'c');

// Query for events in cycle 0 (time 0 to 1)
const state = new State(new TimeSpan(0, 1));
const events = pattern.query(state);

// events will contain:
// [
//   Hap(whole: 0→1/3, part: 0→1/3, value: 'a'),
//   Hap(whole: 1/3→2/3, part: 1/3→2/3, value: 'b'),
//   Hap(whole: 2/3→1, part: 2/3→1, value: 'c')
// ]
```

### Pattern Combinators

**Temporal Combinators:**
- `fast(n)`: Speed up pattern by factor n
- `slow(n)`: Slow down pattern by factor n
- `early(n)`: Shift pattern earlier by n cycles
- `late(n)`: Shift pattern later by n cycles

**Structural Combinators:**
- `sequence(...patterns)`: Play patterns in sequence
- `stack(...patterns)`: Play patterns simultaneously
- `fastcat(...patterns)`: Fast concatenation
- `slowcat(...patterns)`: Slow concatenation

**Applicative Operations:**
- `appLeft(pattern)`: Apply function pattern to value pattern (left structure)
- `appRight(pattern)`: Apply function pattern to value pattern (right structure)
- `appBoth(pattern)`: Apply with both structures

### Monadic Operations

Patterns support monadic bind operations for complex transformations:

```javascript
// bind: Pattern<a> → (a → Pattern<b>) → Pattern<b>
pattern.bind(value => anotherPattern(value))
```

### Value Transformation

**Control Methods:**
Every audio parameter can be set as a method on patterns:

```javascript
pattern.note("c3 e3 g3")  // Set note values
      .s("sawtooth")      // Set synth
      .cutoff(1000)       // Set filter cutoff
      .gain(0.8)          // Set gain
```

These methods are generated by `createParam()` in `controls.mjs`.

### Fraction-Based Timing

All timing uses rational fractions (via `fraction.js`) to avoid floating-point errors:

```javascript
const time = Fraction(1, 3);  // 1/3
const nextTime = time.add(Fraction(1, 3));  // 2/3
```

This ensures perfect rhythmic accuracy even after many cycles.

---

## Mini Notation Parser

### Grammar Overview

The mini notation is defined in `/home/user/strudel/packages/mini/krill.pegjs` using PEG (Parsing Expression Grammar).

**Basic Syntax:**

```
pattern       ::= element+
element       ::= atom | group | polyrythm
atom          ::= word | number | rest
group         ::= '[' pattern ']'
rest          ::= '~'
```

**Examples:**

```javascript
// Sequence
mini("bd sd hh sd")
// → plays bd, sd, hh, sd in order

// Polyrhythm (subdivision)
mini("bd [sd cp]")
// → bd takes first half, sd and cp share second half

// Speed multiplication
mini("bd*2 sd")
// → bd plays twice, sd plays once

// Euclidean rhythm
mini("bd(3,8)")
// → 3 hits distributed evenly over 8 steps

// Rests
mini("bd ~ sd ~")
// → bd, silence, sd, silence
```

### AST to Pattern Conversion

The parser generates an AST which is then converted to Pattern objects in `mini.mjs`:

```javascript
function patternifyAST(ast, code, onEnter, offset = 0) {
  switch (ast.type_) {
    case 'pattern':
      // Process children and apply alignment
      const children = ast.source_.map(child => enter(child));
      return stack(...children);  // or fastcat, slowcat, etc.

    case 'element':
      // Handle atoms, groups, etc.
      return processElement(ast);

    case 'bjorklund':
      // Euclidean rhythms
      return pattern.euclid(pulse, step, rotation);
  }
}
```

### Operators

Mini notation supports various operators:

- `*n`: Multiply (repeat faster)
- `/n`: Divide (slow down)
- `!n`: Replicate
- `@n`: Stretch
- `(p,s)`: Euclidean (p pulses in s steps)
- `(p,s,r)`: Euclidean with rotation

**Example:**
```javascript
mini("bd*2!3")
// → bd*2 (bd plays twice per step) replicated 3 times
```

---

## Transpiler System

### How It Works

The transpiler transforms user-written JavaScript into executable Strudel code:

**File**: `/home/user/strudel/packages/transpiler/transpiler.mjs`

**Process:**

1. **Parse**: Use Acorn to parse JavaScript into AST
2. **Transform**: Walk AST with estree-walker
3. **String Detection**: Find quoted strings and template literals
4. **Wrapping**: Wrap strings with `mini()` function
5. **Location Tracking**: Add source location metadata
6. **Code Generation**: Use escodegen to generate JavaScript

**Example:**

```javascript
// Input
"bd sd hh sd".fast(2)

// Transpiled
mini("bd sd hh sd").withLocation(0, 12, ...).fast(2)
```

### Transformations Applied

**1. String to Mini Notation:**
```javascript
// Detects double-quoted strings
if (isStringWithDoubleQuotes(node)) {
  return this.replace(miniWithLocation(value, node));
}
```

**2. Template Literals:**
```javascript
// Handles backtick strings
if (isBackTickString(node, parent)) {
  return this.replace(miniWithLocation(raw, node));
}
```

**3. Tidal Template Literals:**
```javascript
// Handles Haskell Tidal code in tagged templates
if (isTidalTeplateLiteral(node)) {
  return this.replace(tidalWithLocation(raw, offset));
}
```

**4. Widget Detection:**
```javascript
// Detects slider() and other widget functions
if (isSliderFunction(node)) {
  widgets.push({ type: 'slider', min, max, step });
}
```

**5. Bare samples() Call:**
```javascript
// Wraps samples() with await
if (isBareSamplesCall(node, parent)) {
  return this.replace(withAwait(node));
}
```

### Location Tracking

The transpiler adds source locations to patterns for:
- Code highlighting during playback
- Error reporting
- Interactive widgets

---

## Audio Synthesis (SuperDough)

### Architecture

SuperDough is the default audio engine for Strudel. It handles:
- Sample playback
- Synthesizer voices
- Effects processing
- Polyphony management
- Orbit (bus) routing

**File**: `/home/user/strudel/packages/superdough/superdough.mjs`

### Sound Registration

Sounds are registered in a global sound map:

```javascript
registerSound('bd', async (time, value, onended) => {
  // Trigger sample or synth
  const buffer = await loadBuffer(url);
  playBuffer(buffer, time, value);
});
```

### Sample Loading

Samples can be loaded from:

1. **Sample Map Object:**
```javascript
samples({
  '_base': 'https://example.com/samples/',
  'bd': 'kick.wav',
  'sd': ['snare1.wav', 'snare2.wav']
})
```

2. **JSON URL:**
```javascript
samples('https://example.com/samples.json')
```

3. **GitHub Repository:**
```javascript
samples('github:tidalcycles/dirt-samples')
```

**File**: `/home/user/strudel/packages/superdough/sampler.mjs`

### Synth Engine

Synthesizers use Web Audio oscillators:

```javascript
registerSynthSounds()  // Registers sawtooth, square, triangle, sine

// Usage
superdough({
  s: 'sawtooth',
  note: 'c3',
  cutoff: 1000,
  resonance: 10
}, time, duration)
```

**File**: `/home/user/strudel/packages/superdough/synth.mjs`

### Effects Chain

Each sound goes through:

1. **Source** (sample or oscillator)
2. **Envelope** (ADSR)
3. **Filters** (lowpass, highpass, bandpass)
4. **Distortion** (crush, distort)
5. **Pan**
6. **Orbit Effects** (delay, reverb)
7. **Compressor** (dynamic range)
8. **Output**

### Audio Worklets

SuperDough uses Audio Worklets for low-latency processing:

**File**: `/home/user/strudel/packages/superdough/worklets.mjs`

**Processors:**
- Time-stretching (OLA - Overlap-Add)
- FFT analysis
- Custom DSP effects

### Orbit System

Orbits are like effect buses in a DAW:

```javascript
pattern1.orbit(0)  // Orbit 0 effects
pattern2.orbit(1)  // Orbit 1 effects (independent)
```

Each orbit has its own delay and reverb units.

**Multi-channel Orbits:**
```javascript
setMultiChannelOrbits(true)
// Each orbit gets its own stereo channel pair
```

### Polyphony Management

Default maximum: 128 voices

```javascript
setMaxPolyphony(256)  // Increase to 256
```

Oldest voices are cut when limit is reached.

### Cut Groups

Sounds can cut each other off:

```javascript
pattern.cut(1)  // All sounds in cut group 1 will cut each other
```

Useful for hi-hats and other mutually exclusive sounds.

---

## Website and REPL

### Tech Stack

- **Framework**: Astro 5.x (Static Site Generator + Islands Architecture)
- **UI**: React 19 (for interactive components)
- **Editor**: CodeMirror 6
- **Styling**: TailwindCSS
- **Auth**: Supabase
- **Search**: Algolia DocSearch

**File**: `/home/user/strudel/website/package.json`

### Directory Structure

```
website/
├── src/
│   ├── components/     # React components
│   ├── content/        # MDX documentation
│   ├── docs/          # Additional docs
│   ├── layouts/       # Astro layouts
│   ├── pages/         # Astro pages (routing)
│   ├── repl/          # REPL implementation
│   └── styles/        # Global styles
├── public/            # Static assets
└── astro.config.mjs   # Astro configuration
```

### REPL Implementation

The REPL is built with:

**Key Components:**
- **CodeMirror Editor**: Syntax highlighting, autocomplete
- **Pattern Evaluator**: Transpile and evaluate code
- **Scheduler**: Clock-based event scheduling
- **Audio Context**: Web Audio API integration
- **Visual Feedback**: Code highlighting, error display

**Flow:**

1. User types code in CodeMirror
2. On Ctrl+Enter (or auto-eval):
   - Code is transpiled
   - Pattern is evaluated
   - Pattern is registered with scheduler
3. Scheduler queries pattern for upcoming events
4. Events are sent to audio engine
5. Active code is highlighted

### User Patterns

Users can save patterns to local storage or Supabase:

**File**: `/home/user/strudel/website/src/user_pattern_utils.mjs`

Features:
- Save/load patterns
- Share via URL
- Fork patterns
- Version history (if logged in)

### Settings

User preferences stored in localStorage via nanostores:

**File**: `/home/user/strudel/website/src/settings.mjs`

Settings:
- Audio device selection
- Keymap (default, vim, emacs, vscode)
- Font size
- Auto-format
- Theme

### Desktop App

Strudel has a desktop version built with Tauri:

**Directory**: `/home/user/strudel/src-tauri/`

Advantages:
- Better audio latency
- File system access
- Native performance

---

## How Everything Connects

### Full Stack Flow

```
┌──────────────────────────────────────────────────────┐
│ 1. USER WRITES CODE                                  │
│    "bd sd hh sd".fast(2).s("sawtooth")              │
└──────────────────────────────────────────────────────┘
                      ↓
┌──────────────────────────────────────────────────────┐
│ 2. TRANSPILER                                        │
│    - Parse with Acorn                                │
│    - Detect strings → wrap with mini()              │
│    - Add location tracking                           │
│    - Generate JavaScript                             │
└──────────────────────────────────────────────────────┘
                      ↓
┌──────────────────────────────────────────────────────┐
│ 3. EVALUATION                                        │
│    - mini() parses "bd sd hh sd"                    │
│    - Creates Pattern object                          │
│    - .fast(2) transforms pattern                     │
│    - .s("sawtooth") sets control                    │
└──────────────────────────────────────────────────────┘
                      ↓
┌──────────────────────────────────────────────────────┐
│ 4. SCHEDULING (Clock)                                │
│    - Clock ticks every ~100ms                        │
│    - Queries pattern for time window                 │
│    - Pattern returns Hap events                      │
└──────────────────────────────────────────────────────┘
                      ↓
┌──────────────────────────────────────────────────────┐
│ 5. EVENT PROCESSING                                  │
│    - Events scheduled with Web Audio clock           │
│    - Hap values converted to audio parameters        │
└──────────────────────────────────────────────────────┘
                      ↓
┌──────────────────────────────────────────────────────┐
│ 6. AUDIO RENDERING (SuperDough)                      │
│    - Load sample or create oscillator                │
│    - Apply envelope, filters, effects                │
│    - Route through orbit                             │
│    - Output to speakers                              │
└──────────────────────────────────────────────────────┘
```

### Clock System

**File**: `/home/user/strudel/packages/core/zyklus.mjs`

The clock is the heartbeat of Strudel:

```javascript
const clock = createClock(
  getTime,           // () => AudioContext.currentTime
  callback,          // (phase, duration, tick, t) => {...}
  duration = 0.05,   // Cycle duration (20 cycles/sec default)
  interval = 0.1,    // Callback interval
  overlap = 0.1      // Lookahead overlap
)
```

**How It Works:**

1. Clock ticks at regular intervals (default 100ms)
2. Each tick looks ahead by `interval + overlap`
3. Queries patterns for events in that window
4. Schedules events with Web Audio precise timing
5. Events are triggered at exact times

**Benefits:**
- Precise timing via Web Audio clock
- Lookahead prevents glitches
- Compensates for JavaScript timing jitter

### Pattern Evaluation

**File**: `/home/user/strudel/packages/core/evaluate.mjs`

```javascript
export function evaluate(code, transpiler) {
  // 1. Transpile code
  const transpiled = transpiler(code);

  // 2. Create async function
  const func = new Function('return (async () => {' + transpiled + '})()');

  // 3. Execute in context with Strudel globals
  const result = func.call(context);

  // 4. Return pattern
  return result;
}
```

### REPL Integration

The REPL connects all pieces:

**Pattern Registration:**
```javascript
// When user evaluates code:
const pattern = await evaluate(code, transpiler);

// Register with scheduler:
scheduler.setPattern(pattern, {
  id: 'user-pattern',
  autolink: true  // Link with clock
});
```

**Scheduler Callback:**
```javascript
function onEvent(hap, deadline, duration, cps) {
  // Extract audio parameters from hap.value
  const params = {
    s: hap.value.s || 'sine',
    note: hap.value.note,
    gain: hap.value.gain || 0.5,
    ...hap.value
  };

  // Trigger audio
  superdough(params, deadline, duration);
}
```

---

## Key Files Reference

### Core Engine Files

| File | Lines | Purpose |
|------|-------|---------|
| `/packages/core/pattern.mjs` | 3000+ | Pattern class, all combinators and transformations |
| `/packages/core/controls.mjs` | 1700+ | Audio control parameters and methods |
| `/packages/core/hap.mjs` | 184 | Event representation |
| `/packages/core/timespan.mjs` | 118 | Time interval mathematics |
| `/packages/core/state.mjs` | 29 | Query state container |
| `/packages/core/fraction.mjs` | 150+ | Fraction utilities and operations |
| `/packages/core/zyklus.mjs` | 55 | Clock implementation |
| `/packages/core/signal.mjs` | 600+ | Continuous signal patterns |
| `/packages/core/euclid.mjs` | 200+ | Euclidean rhythm generation |

### Parser Files

| File | Lines | Purpose |
|------|-------|---------|
| `/packages/mini/krill.pegjs` | 600+ | PEG grammar for mini notation |
| `/packages/mini/krill-parser.js` | 3300+ | Generated parser (don't edit directly) |
| `/packages/mini/mini.mjs` | 300+ | AST to Pattern conversion |

### Transpiler Files

| File | Lines | Purpose |
|------|-------|---------|
| `/packages/transpiler/transpiler.mjs` | 400+ | Code transformation engine |
| `/packages/transpiler/index.mjs` | 6 | Export wrapper |

### Audio Files

| File | Lines | Purpose |
|------|-------|---------|
| `/packages/superdough/superdough.mjs` | 600+ | Main audio engine |
| `/packages/superdough/sampler.mjs` | 350+ | Sample loading and playback |
| `/packages/superdough/synth.mjs` | 300+ | Synthesizer implementation |
| `/packages/superdough/worklets.mjs` | 700+ | Audio Worklet processors |
| `/packages/superdough/helpers.mjs` | 400+ | Web Audio utilities |

### Website Files

| File | Purpose |
|------|---------|
| `/website/astro.config.mjs` | Astro configuration |
| `/website/src/repl/` | REPL implementation |
| `/website/src/settings.mjs` | User settings management |
| `/website/src/user_pattern_utils.mjs` | Pattern save/load |

### Configuration Files

| File | Purpose |
|------|---------|
| `/package.json` | Monorepo package definition |
| `/pnpm-workspace.yaml` | pnpm workspace configuration |
| `/lerna.json` | Lerna monorepo config |
| `/vitest.config.mjs` | Test configuration |

---

## Code Examples

### Example 1: Simple Pattern

```javascript
// Create a simple drum pattern
"bd sd hh sd"
  .bank("RolandTR909")  // Use TR-909 samples
  .fast(2)              // Play twice as fast
  .gain(0.8)            // Set volume
```

**Breakdown:**
1. `"bd sd hh sd"` → Transpiled to `mini("bd sd hh sd")`
2. Mini parser creates sequence of 4 sounds
3. `.bank()` adds bank prefix to each sound
4. `.fast(2)` speeds up by 2x
5. `.gain(0.8)` sets volume to 80%

### Example 2: Polyrhythm

```javascript
// Create complex polyrhythm
stack(
  "bd*2",           // Bass drum on every half beat
  "~ sd",           // Snare on beat 2
  "hh*8"            // Hi-hat 8 times per cycle
).slow(2)           // Slow down entire stack
```

**Breakdown:**
1. Three patterns stacked (played simultaneously)
2. Each has different subdivision
3. `slow(2)` makes entire pattern take 2 cycles

### Example 3: Generative Pattern

```javascript
// Random note selection from scale
note("c d e f g a b".segment(7))  // Split into 7 segments
  .scale("C:minor")                // Apply minor scale
  .struct("x(3,8)")                // Euclidean rhythm structure
  .s("sawtooth")                   // Synth waveform
  .cutoff(sine.range(500, 2000))   // Modulate filter
  .room(0.5)                       // Add reverb
```

**Breakdown:**
1. Scale degrees selected from sequence
2. `.scale()` converts to actual notes
3. `.struct()` applies Euclidean rhythm (3 hits in 8 steps)
4. `.cutoff()` uses continuous sine wave for modulation
5. Effects applied

### Example 4: Pattern Transformation

```javascript
// Create evolving pattern
"c3 e3 g3"
  .every(4, x => x.rev())        // Reverse every 4th cycle
  .off(1/8, x => x.add(7))       // Add harmony offset by 1/8
  .iter(4)                       // Rotate pattern over 4 cycles
  .s("sawtooth")
  .lpf(800)
```

**Breakdown:**
1. Base pattern: C major triad
2. `.every(4, rev)` reverses every 4th cycle
3. `.off(1/8, add(7))` creates harmony 1/8 cycle later, +7 semitones
4. `.iter(4)` rotates starting point each cycle
5. Audio controls applied

### Example 5: Custom Pattern Function

```javascript
// Define reusable pattern function
const arpeggio = (notes, rate = 4) =>
  notes
    .fast(rate)
    .off(1/4, x => x.add(12))     // Octave up
    .s("sine");

// Use it
arpeggio("c e g", 8)
  .lpf(1200)
  .room(0.3)
```

**Breakdown:**
1. Function takes notes and rate
2. Speeds up and adds octave layer
3. Can be reused with different inputs

### Example 6: Using Mini Notation Features

```javascript
// Complex mini notation
"bd(3,8)!2 sd*2 [hh hh:2]*4 ~"
  .bank("RolandTR909")
```

**Breakdown:**
1. `bd(3,8)` → 3 bass drums in 8 steps (Euclidean)
2. `!2` → Replicate the Euclidean pattern twice
3. `sd*2` → Snare plays twice
4. `[hh hh:2]` → Group with two hi-hat variations
5. `*4` → Repeat group 4 times
6. `~` → Rest

### Example 7: Multiple Outputs

```javascript
// Send to different outputs simultaneously
const melody = "c3 e3 g3".s("sawtooth");

stack(
  melody.midi(),                    // Send to MIDI
  melody.scope(),                   // Show on oscilloscope
  melody.osc(),                     // Send via OSC
  melody.webdough()                 // Play with SuperDough
)
```

**Breakdown:**
Pattern is sent to multiple outputs at once.

### Example 8: Audio Parameters

```javascript
// All audio parameters in one pattern
note("c3 e3 g3")
  .s("sawtooth")           // Waveform
  .gain(0.8)               // Volume
  .pan(sine.slow(4))       // Auto-pan with LFO
  .cutoff(1000)            // Low-pass filter
  .resonance(10)           // Filter resonance
  .attack(0.01)            // Envelope attack
  .decay(0.1)              // Envelope decay
  .sustain(0.5)            // Envelope sustain
  .release(0.5)            // Envelope release
  .delay(0.3)              // Delay mix
  .room(0.5)               // Reverb mix
  .orbit(0)                // Effect bus
```

### Example 9: Pattern Variables

```javascript
// Store patterns in variables
const kick = "bd*2".gain(0.9);
const snare = "~ sd".gain(0.7);
const hats = "hh*8".gain(0.5);

// Combine
stack(kick, snare, hats)
  .bank("RolandTR909")
  .cpm(120)  // 120 cycles per minute
```

### Example 10: Loading Custom Samples

```javascript
// Load custom samples
await samples({
  '_base': 'https://mysite.com/samples/',
  'kick': 'my-kick.wav',
  'snare': ['snare-1.wav', 'snare-2.wav', 'snare-3.wav']
});

// Use them
"kick snare:1 kick snare:2"
  .gain(0.8)
```

**Breakdown:**
1. Samples loaded from URL
2. Multiple variations for snare
3. Use with `:n` to select variation

---

## Development Workflow

### Setup

```bash
# Clone repository
git clone https://github.com/tidalcycles/strudel.git
cd strudel

# Install dependencies
pnpm install

# Run development server
pnpm dev
```

### Building

```bash
# Build all packages
pnpm build

# Build specific package
cd packages/core
pnpm build
```

### Testing

```bash
# Run all tests
pnpm test

# Run tests with UI
pnpm test-ui

# Run tests with coverage
pnpm test-coverage

# Run benchmarks
pnpm bench
```

### Documentation Generation

```bash
# Generate JSDoc documentation
pnpm jsdoc

# Generate JSON documentation
pnpm jsdoc-json

# Report undocumented functions
pnpm report-undocumented
```

### Code Quality

```bash
# Lint code
pnpm lint

# Format code
pnpm codeformat

# Check formatting
pnpm format-check

# Run all checks
pnpm check
```

---

## Architecture Decisions

### Why Patterns are Functions

Patterns are functions (not data structures) because:
- **Lazy evaluation**: Events only generated when needed
- **Infinite patterns**: Can represent infinitely long patterns
- **Composability**: Functions compose naturally
- **Memory efficient**: Don't store all events

### Why Rational Fractions

Using `fraction.js` instead of floating-point:
- **Perfect precision**: No rounding errors
- **Exact divisions**: 1/3 is exactly 1/3, not 0.333...
- **Reliable equality**: Can safely compare times

### Why Transpiler

The transpiler enables:
- **Concise syntax**: `"bd sd"` instead of `mini("bd sd")`
- **Source tracking**: Know which code is playing
- **Interactive widgets**: Auto-detect UI controls
- **Better errors**: Point to exact source location

### Why Monorepo

Benefits of the monorepo structure:
- **Shared dependencies**: All packages use same versions
- **Easy development**: Work across packages seamlessly
- **Atomic commits**: Change multiple packages at once
- **Consistent tooling**: Same build/test/lint everywhere

---

## Performance Considerations

### Pattern Query Optimization

- Patterns are queried for small time windows (not entire piece)
- Events outside query window are never generated
- Caching can be added for expensive patterns

### Audio Scheduling

- Uses Web Audio's precise timing
- Lookahead scheduling prevents glitches
- Events scheduled ahead of playback time

### Memory Management

- Samples loaded once and cached
- Audio nodes reused when possible
- Polyphony limiting prevents memory leaks

### Web Worker Potential

Future optimization could use Web Workers for:
- Pattern evaluation
- Sample loading
- Non-audio-thread processing

---

## Future Development

### Potential Enhancements

1. **Better Type Safety**: TypeScript migration
2. **Improved Performance**: Web Worker pattern evaluation
3. **More Outputs**: More synthesizers, effects, visualizations
4. **Better Mobile**: Touch-optimized interface
5. **Collaboration**: Multi-user live coding
6. **Recording**: Built-in audio recording
7. **Plugin System**: User-extensible architecture

### Community Contributions

Strudel is open source (AGPL-3.0). Contributions welcome:
- Bug fixes
- New features
- Documentation
- Examples
- Tutorials

See `/home/user/strudel/CONTRIBUTING.md` for guidelines.

---

## Resources

### Official Links

- **Website**: https://strudel.cc
- **Documentation**: https://strudel.cc/learn
- **Repository**: https://github.com/tidalcycles/strudel
- **Discord**: https://discord.com/invite/HGEdXmRkzT
- **Forum**: https://club.tidalcycles.org/

### Related Projects

- **TidalCycles**: Original Haskell implementation
- **Vortex**: Python port of Tidal
- **Hydra**: Video synthesizer
- **SuperCollider**: Audio synthesis platform
- **SuperDirt**: TidalCycles audio engine

### Learning Resources

- Strudel Tutorial: https://strudel.cc/learn
- TidalCycles Docs: https://tidalcycles.org
- Mini Notation Guide: https://strudel.cc/learn/mini-notation
- Technical Blog: https://loophole-letters.vercel.app/strudel

---

## Glossary

- **Pattern**: A function that generates events over time
- **Hap**: A single event with timing and value
- **TimeSpan**: A time interval from begin to end
- **Cycle**: One iteration of a pattern (typically 1 beat or bar)
- **Mini Notation**: Concise syntax for rhythmic patterns
- **Transpiler**: Transforms user code into executable code
- **SuperDough**: Strudel's audio engine
- **Orbit**: Effect bus for routing sounds
- **Control**: Audio parameter (note, gain, filter, etc.)
- **Query**: Requesting events from a pattern for a time window
- **Combinator**: Function that combines or transforms patterns
- **Euclidean Rhythm**: Evenly distributed hits (e.g., 3 in 8)
- **REPL**: Read-Eval-Print Loop (interactive coding environment)

---

*Last updated: 2025-11-16*
*Strudel Version: 0.5.0*
*Documentation generated from codebase exploration*
