# Strudel Simplification Opportunities

This document identifies packages, features, dependencies, and configurations that could be removed or simplified to create a minimal, lean version of Strudel while preserving core live coding music functionality.

## Executive Summary

Strudel currently has **27 packages** with extensive integrations, a complex web UI, and comprehensive documentation. A minimal version could reduce this to **5-7 core packages** by removing:
- 10 integration packages (MIDI, OSC, CSound, Hydra, gamepad, motion, serial, MQTT, soundfonts, desktopbridge)
- 3 experimental/specialized packages (tidal, hs2js, embed)
- 1 large optional package (xen - 1.2MB)
- Simplified REPL with reduced UI features
- Streamlined documentation site

**Estimated reduction: ~60-70% of packages, ~50-60% of UI complexity**

---

## 1. Packages to Remove

### 1.1 Hardware/Protocol Integration Packages (HIGH PRIORITY)

#### @strudel/midi
- **What**: MIDI input/output support via WebMIDI API
- **Dependencies**: webmidi (3.1.12), @strudel/core, @strudel/webaudio
- **Why cut**: Specialized use case, adds complexity for hardware integration
- **Impact**: Users cannot send/receive MIDI messages or control external hardware synthesizers
- **Use case**: ~10-20% of users need MIDI

#### @strudel/osc
- **What**: OSC (Open Sound Control) messaging for communication with external software
- **Dependencies**: osc-js (2.4.1)
- **Why cut**: Advanced feature for integration with SuperCollider, Max/MSP, etc.
- **Impact**: Cannot communicate with external live coding environments
- **Use case**: ~5-15% of users, primarily advanced live coders

#### @strudel/serial
- **What**: Web Serial API for Arduino/hardware communication
- **Dependencies**: @strudel/core
- **Why cut**: Very niche use case for physical computing
- **Impact**: Cannot control Arduino or other serial devices
- **Use case**: <5% of users

#### @strudel/mqtt
- **What**: MQTT protocol for IoT messaging
- **Dependencies**: paho-mqtt (1.1.0)
- **Why cut**: Extremely niche, IoT-specific integration
- **Impact**: Cannot integrate with MQTT-based IoT systems
- **Use case**: <5% of users

#### @strudel/gamepad
- **What**: Game controller input support
- **Dependencies**: @strudel/core
- **Size**: 24K
- **Why cut**: Novel but non-essential input method
- **Impact**: Cannot use game controllers to control patterns
- **Use case**: <10% of users, mostly for performances/demos

#### @strudel/motion
- **What**: Device motion/orientation sensor support
- **Dependencies**: @strudel/core
- **Size**: 29K
- **Why cut**: Mobile-specific, performance-oriented feature
- **Impact**: Cannot use phone accelerometer/gyroscope for control
- **Use case**: <10% of users, mobile performers

**Total savings: 6 packages removed, reduced complexity in hardware/protocol integration**

### 1.2 External Audio Engine Integrations

#### @strudel/csound
- **What**: CSound audio synthesis engine integration
- **Dependencies**: @csound/browser (6.18.7), @strudel/core, @strudel/webaudio
- **Size**: 78K
- **Why cut**: Alternative audio engine, redundant with superdough
- **Impact**: Cannot use CSound synthesis, but superdough covers most needs
- **Use case**: ~5-10% of users familiar with CSound

#### @strudel/hydra
- **What**: Hydra visual synthesizer integration for visuals
- **Dependencies**: hydra-synth (1.3.29), @strudel/core, @strudel/draw
- **Size**: 8.5K
- **Why cut**: Visual feature, not core to audio live coding
- **Impact**: No visual feedback via Hydra, but other visualization options exist
- **Use case**: ~10-20% of users who want visuals

**Total savings: 2 packages removed, focus on audio-only**

### 1.3 Sound Source Extensions

#### @strudel/soundfonts
- **What**: Soundfont (.sf2) file support for realistic instruments
- **Dependencies**: sfumato (0.1.2), soundfont2 (0.5.0), @strudel/core, @strudel/webaudio
- **Size**: 127K
- **Why cut**: Large dependency, niche use case (orchestral sounds)
- **Impact**: Cannot load soundfonts, but samples and synths still work
- **Use case**: ~10-15% of users wanting orchestral/realistic sounds

**Total savings: 1 package removed, 127K+**

### 1.4 Music Theory Extensions

#### @strudel/xen
- **What**: Xenharmonic/microtonal music support (non-12-tone scales)
- **Dependencies**: @strudel/core
- **Size**: 1.2MB (!!)
- **Why cut**: Extremely specialized, largest package by far
- **Impact**: Cannot use microtonal scales beyond 12-tone equal temperament
- **Use case**: <5% of users, experimental musicians
- **Note**: This is the single largest package - huge size win

**Total savings: 1 package removed, 1.2MB saved**

### 1.5 Experimental/Incomplete Packages

#### @strudel/tidal (private)
- **What**: Experimental Haskell Tidal syntax interpreter
- **Dependencies**: hs2js, @strudel/core, @strudel/mini
- **Size**: 7.5K
- **Why cut**: Marked as private/experimental, not production-ready
- **Impact**: Cannot write Haskell Tidal syntax directly (but mini-notation still works)
- **Use case**: Experimental feature

#### hs2js (private)
- **What**: Haskell-to-JavaScript interpreter using tree-sitter
- **Dependencies**: web-tree-sitter (0.24.7), tree-sitter-haskell
- **Size**: 71K + WASM files
- **Why cut**: Experimental, supports @strudel/tidal which is also experimental
- **Impact**: No Haskell syntax support
- **Use case**: Experimental feature

**Total savings: 2 packages removed, experimental features**

### 1.6 Platform-Specific Packages

#### @strudel/desktopbridge (private)
- **What**: Bridge between JavaScript and Tauri (Rust) for desktop app
- **Dependencies**: @tauri-apps/api, @strudel/core
- **Size**: 11K
- **Why cut**: Only needed for desktop app, web version doesn't need it
- **Impact**: Cannot build desktop app version
- **Use case**: Desktop app users only

#### @strudel/embed
- **What**: Web component for embedding Strudel in iframes
- **Size**: 8K
- **Why cut**: Convenience feature for embedding in other sites
- **Impact**: Harder to embed Strudel in other websites
- **Use case**: ~5-10% of users embedding in blogs/tutorials

**Total savings: 2 packages removed for web-focused minimal version**

### 1.7 Development/Tooling Packages

#### @strudel/sampler
- **What**: Local sample server for development
- **Dependencies**: cowsay (!)
- **Size**: 8.5K
- **Why cut**: Development tool, users can use remote sample URLs
- **Impact**: Cannot run local sample server
- **Use case**: Developers only

#### @strudel/reference
- **What**: Headless reference documentation generator
- **Size**: 6.5K
- **Why cut**: Build-time tool, not needed at runtime
- **Impact**: Reference must be pre-generated
- **Use case**: Build process only

**Total savings: 2 packages removed from runtime**

---

## 2. Core Packages to Keep

These packages are essential for minimal live coding functionality:

### 2.1 Essential Core (KEEP)

#### @strudel/core
- **Size**: 340K (largest, but essential)
- **Why keep**: Pattern library, fundamental algorithms (8k+ lines)
- **Modules**: pattern.mjs (3.3k lines), controls.mjs (1.8k lines), signal.mjs, etc.
- **Possible simplification**: Could review controls.mjs for rarely-used controls

#### @strudel/mini
- **Size**: 172K
- **Why keep**: Mini-notation parser (core syntax for Tidal/Strudel)
- **Dependencies**: peggy parser generator
- **Critical**: This is the primary input method for most users

#### @strudel/transpiler
- **Size**: 22K
- **Why keep**: Converts user JavaScript to evaluatable patterns
- **Dependencies**: acorn, escodegen, estree-walker
- **Critical**: Required for the JavaScript syntax

#### @strudel/webaudio
- **Size**: 17K
- **Why keep**: Web Audio API integration, core audio output
- **Dependencies**: superdough, @strudel/core, @strudel/draw

#### superdough
- **Size**: 138K
- **Why keep**: Main audio engine (synth + sampler)
- **Dependencies**: nanostores
- **Note**: Inspired by SuperDirt, this is the sound engine

### 2.2 Enhanced Core (CONSIDER KEEPING)

#### @strudel/tonal
- **Size**: 65K
- **Why keep**: Music theory functions (scales, chords, note names)
- **Dependencies**: @tonaljs/tonal, chord-voicings, webmidi
- **Decision**: Keep for musical functionality, but optional for absolute minimal
- **Note**: Provides user-friendly musical features

#### @strudel/draw
- **Size**: 42K
- **Why keep**: Visual feedback (pianoroll, waveforms)
- **Decision**: Useful for learning/debugging, but optional
- **Impact if removed**: No visual pattern representation

#### @strudel/codemirror
- **Size**: 131K
- **Why keep**: Code editor with syntax highlighting, autocomplete
- **Dependencies**: Large (many @codemirror/* packages)
- **Decision**: Could replace with simpler textarea in minimal version
- **Alternative**: Simple textarea or Monaco editor (lighter?)

---

## 3. Features to Simplify or Remove

### 3.1 REPL UI Features (from website/src/settings.mjs and components)

#### Multiple Keybinding Modes
- **Current**: CodeMirror, Vim, Emacs, VSCode keybindings
- **Dependencies**: @replit/codemirror-vim, @replit/codemirror-emacs, @replit/codemirror-vscode-keymap
- **Simplification**: Keep only default CodeMirror bindings
- **Impact**: Power users lose familiar keybindings
- **Savings**: 3 dependencies removed

#### Theme System
- **Current**: Customizable themes, theme switching
- **Simplification**: Single dark theme only
- **Impact**: Users cannot customize appearance
- **Savings**: Theme management code removed

#### Panel System Complexity
- **Current**: Multiple tabs (Welcome, Console, Files, Sounds, Settings, Patterns, Reference)
- **Current**: Panel positioning (left/right/bottom), pinning, hover triggers
- **Simplification**: Single bottom panel with Code + Sounds only
- **Impact**: Less UI flexibility, removed features:
  - Files tab (local file management)
  - Settings tab (move to modal)
  - Welcome tab (show once on first load)
  - Reference tab (link to external docs)
  - Patterns tab (community pattern browser)
- **Savings**: 5 complex components removed

#### Code Editor Features
- **Current**: Line numbers, bracket matching, bracket closing, autocomplete, tooltips, line wrapping, pattern highlighting, active line highlighting
- **Simplification**: Keep only line numbers, bracket matching, syntax highlighting
- **Remove**: Autocomplete, tooltips, pattern highlighting (complex features)
- **Impact**: Less helpful for beginners
- **Savings**: Simpler editor configuration

#### Audio Settings
- **Current**: Multiple audio devices, engine target selection (webaudio/osc), multi-channel orbits, max polyphony settings
- **Simplification**: Single default audio device, webaudio only, fixed settings
- **Impact**: Cannot choose audio device or use OSC
- **Savings**: Remove audio device selector components

#### Pattern Management
- **Current**: User patterns storage (localStorage), community patterns, pattern filtering
- **Simplification**: Remove user pattern storage, just have examples
- **Impact**: Cannot save patterns locally (use browser bookmarks or external files)
- **Savings**: Remove user_pattern_utils.mjs, pattern storage logic

#### Advanced Settings
- **Current**: CSS animation toggle, flash highlighting, sync mode, zen mode, button row toggle
- **Simplification**: Remove all toggles, use sensible defaults
- **Impact**: Less customization
- **Savings**: Simpler settings system

### 3.2 Website/Documentation Complexity

#### Multi-language Support
- **Current**: English + German, i18n setup
- **Location**: website/src/pages/de/*, config.ts KNOWN_LANGUAGES
- **Simplification**: English only
- **Impact**: German users need to use English
- **Savings**: Remove /de directory, i18n logic

#### Documentation Sections
- **Current**: 7+ major sections (Workshop, Making Sound, More, Pattern Functions, Understand, Development, Presentation)
- **Current**: 70+ pages total
- **Simplification**: Keep only:
  - Quick Start Guide (1 page)
  - Basic Tutorial (3-4 pages)
  - Function Reference (generated)
- **Remove**:
  - Extensive workshop materials
  - Technical manual (for advanced users)
  - Recipes section
  - Blog
  - Showcase
  - Bakery (community patterns)
- **Impact**: Less learning material (link to full docs)
- **Savings**: ~60+ pages removed, simpler navigation

#### Search Integration
- **Current**: Algolia DocSearch integration
- **Dependencies**: @algolia/client-search, @docsearch/react, @docsearch/css
- **Simplification**: Remove search, use browser find
- **Impact**: Cannot search docs (but fewer docs to search)
- **Savings**: 3 dependencies removed

#### PWA Features
- **Current**: Progressive Web App, offline support, service workers
- **Dependencies**: @vite-pwa/astro, workbox-window
- **Simplification**: Regular website only, no offline
- **Impact**: Cannot use offline or install as app
- **Savings**: 2 dependencies, simpler deployment

#### Social Features
- **Current**: Supabase integration for pattern sharing
- **Dependencies**: @supabase/supabase-js
- **Simplification**: Remove social/sharing features
- **Impact**: Cannot share patterns with community
- **Savings**: 1 large dependency, backend infrastructure

#### Video Embeds
- **Current**: YouTube embeds, video tutorials
- **Dependencies**: @astro-community/astro-embed-youtube, react-lite-youtube-embed
- **Simplification**: Simple links to videos instead of embeds
- **Impact**: Less rich documentation
- **Savings**: 2 dependencies

#### UI Components
- **Current**: Headless UI components, Hero Icons
- **Dependencies**: @headlessui/react, @heroicons/react
- **Simplification**: Plain HTML/CSS components
- **Impact**: Less polished UI
- **Savings**: 2 dependencies

#### Advanced REPL Components
- **Current**: Claviature (piano keyboard), PitchSlider, Showcase
- **Simplification**: Remove visual widgets
- **Impact**: Less interactive learning tools
- **Savings**: 3+ complex components

---

## 4. Dependencies to Cut

### 4.1 External Service Dependencies

```
REMOVE:
- @supabase/supabase-js          # Pattern sharing backend
- @algolia/client-search         # Documentation search
- @docsearch/react              # Search UI
- @docsearch/css                # Search styles
```

### 4.2 Editor Enhancement Dependencies

```
REMOVE:
- @replit/codemirror-vim        # Vim keybindings
- @replit/codemirror-emacs      # Emacs keybindings
- @replit/codemirror-vscode-keymap  # VSCode keybindings

KEEP:
- @codemirror/state             # Core editor state
- @codemirror/view              # Core editor view
- @codemirror/lang-javascript   # JavaScript syntax
- @codemirror/commands          # Basic commands
```

### 4.3 Audio Engine Dependencies

```
REMOVE:
- @csound/browser               # CSound engine (78K pkg)
- hydra-synth                   # Hydra visuals
- webmidi (from @strudel/tonal) # MIDI support (duplicate)
- sfumato + soundfont2          # Soundfont support
- osc-js                        # OSC protocol
- paho-mqtt                     # MQTT protocol

KEEP:
- nanostores                    # Lightweight state (superdough)
- fraction.js                   # Fraction math (@strudel/core)
```

### 4.4 UI/Framework Dependencies

```
REMOVE:
- @headlessui/react             # UI component library
- @heroicons/react              # Icon set
- @astro-community/astro-embed-youtube  # Video embeds
- react-lite-youtube-embed      # Lightweight YouTube
- @vite-pwa/astro              # PWA support
- workbox-window               # Service worker
- react-hook-inview            # Intersection observer hook

SIMPLIFY:
- React (keep but minimal usage)
- tailwindcss (consider plain CSS)
```

### 4.5 Build Tool Dependencies

```
CONSIDER REMOVING:
- @tauri-apps/cli               # Desktop app builds
- lerna                         # Monorepo management (use pnpm workspaces)

KEEP:
- vite                          # Build tool
- vitest                        # Testing
- prettier                      # Code formatting
- eslint                        # Linting
```

### 4.6 Documentation Dependencies

```
REMOVE:
- rehype-* plugins              # Markdown processing
- remark-toc                    # Table of contents
- @astrojs/rss                 # RSS feed
- @tailwindcss/typography      # Prose styling (less docs)

KEEP (minimal):
- @astrojs/mdx                 # Basic markdown
- @astrojs/react               # React integration
```

---

## 5. Configuration to Streamline

### 5.1 Build Configuration

**Current complexity:**
- Lerna for monorepo management
- pnpm workspaces
- 27+ package.json files
- Individual package builds
- Astro for website

**Simplifications:**
```
BEFORE: 27 packages
AFTER:  ~7 packages (core, mini, transpiler, webaudio, superdough, web, repl)

REMOVE:
- packages/csound/
- packages/gamepad/
- packages/hydra/
- packages/midi/
- packages/motion/
- packages/mqtt/
- packages/osc/
- packages/serial/
- packages/soundfonts/
- packages/xen/
- packages/tidal/
- packages/hs2js/
- packages/desktopbridge/
- packages/embed/
- packages/sampler/
- packages/reference/

CONSIDER MERGING:
- @strudel/web could include @strudel/repl
- Flatten package structure for minimal version
```

### 5.2 Settings Configuration (settings.mjs)

**Current: 24 settings**
```javascript
// REMOVE these settings:
keybindings            // Just use default
theme                  // Single theme
fontFamily            // Fixed font
isAutoCompletionEnabled  // Remove feature
isTooltipEnabled      // Remove feature
soundsFilter          // Simplified sound browser
patternFilter         // Remove pattern browser
isPanelPinned         // Simplified panel
togglePanelTrigger    // Always click
userPatterns          // Remove storage
audioEngineTarget     // WebAudio only
isButtonRowHidden     // Always show
isCSSAnimationDisabled // Always enabled
multiChannelOrbits    // Remove feature

// KEEP these settings:
isLineNumbersDisplayed
isBracketMatchingEnabled
isBracketClosingEnabled
isFlashEnabled
fontSize
panelPosition
isPanelOpen
isZen (maybe)
maxPolyphony
```

### 5.3 Website Configuration (config.ts)

**Current:**
- Multi-language sidebar definitions
- 7+ documentation sections
- Algolia search config
- Community Discord links
- GitHub edit URLs

**Simplified:**
```typescript
export const SIDEBAR = {
  en: {
    'Quick Start': [
      { text: 'Installation', link: 'quick-start' },
      { text: 'First Pattern', link: 'first-pattern' },
    ],
    'Tutorial': [
      { text: 'Sounds', link: 'tutorial/sounds' },
      { text: 'Notes', link: 'tutorial/notes' },
      { text: 'Effects', link: 'tutorial/effects' },
    ],
    'Reference': [
      { text: 'All Functions', link: 'reference' },
    ],
  },
};
// Remove: Algolia, multi-language, Open Graph complexity
```

### 5.4 Astro Configuration

**Simplifications:**
- Remove MDX plugins (just basic markdown)
- Remove PWA plugin
- Remove RSS feed generation
- Simplify Tailwind config (or remove for plain CSS)
- Remove i18n routing

### 5.5 TypeScript/ESLint Configuration

**Current:**
- Multiple ESLint configs
- TypeScript configs in multiple packages

**Simplified:**
- Single root ESLint config
- Minimal TypeScript (or pure JavaScript)

---

## 6. UI Components to Simplify

### 6.1 Header Component

**Current features:**
- Logo
- Navigation menu
- Settings button
- Zen mode toggle
- Audio device selector
- Pattern save/load
- Share button

**Simplified:**
```
KEEP:
- Logo
- Play/Stop buttons
- Simple menu (collapse to hamburger)

REMOVE:
- Advanced settings
- Audio device selector
- Share button
- Pattern management UI
```

### 6.2 Panel Component

**Current:**
- Multi-tab system (7 tabs)
- Resizable panels
- Drag-to-reposition
- Pin/unpin
- Hover/click triggers

**Simplified:**
```
KEEP:
- Single bottom panel
- Code output display
- Basic sound browser

REMOVE:
- Multiple tabs
- Positioning system
- Pin/hover features
- Files tab
- Settings tab (move to modal)
- Community patterns tab
```

### 6.3 Editor Component (Textbox)

**Current:**
- Full CodeMirror setup
- Multiple keybinding modes
- Advanced extensions
- Pattern highlighting
- Tooltips
- Autocomplete

**Simplified:**
```
KEEP:
- Basic CodeMirror
- Syntax highlighting
- Line numbers
- Bracket matching

REMOVE:
- Alternative keybindings (Vim/Emacs/VSCode)
- Autocomplete system
- Tooltip system
- Complex pattern highlighting
```

### 6.4 Visualization Components

**Current:**
- Pianoroll display
- Waveform display
- Hydra integration
- Mini notation preview
- Pattern cycle visualization

**Simplified:**
```
KEEP:
- Basic pattern output display
- Simple activity indicator

REMOVE OR SIMPLIFY:
- Pianoroll (nice-to-have)
- Waveform (performance cost)
- Hydra (separate package)
- Complex visualizations
```

---

## 7. Minimal Version Architecture

### 7.1 Package Structure

```
strudel-minimal/
├── packages/
│   ├── core/           # Pattern library (KEEP)
│   ├── mini/           # Mini notation (KEEP)
│   ├── transpiler/     # Code transpiler (KEEP)
│   ├── webaudio/       # Web Audio (KEEP)
│   ├── superdough/     # Audio engine (KEEP)
│   ├── tonal/          # Music theory (OPTIONAL)
│   └── web/            # Bundled minimal version
├── website/
│   ├── pages/
│   │   ├── index.astro        # Landing/REPL
│   │   ├── quick-start.md     # Quick start
│   │   ├── tutorial/          # 3-4 tutorial pages
│   │   └── reference.md       # Function reference
│   └── components/
│       ├── Editor.jsx         # Simple code editor
│       ├── Controls.jsx       # Play/stop/tempo
│       └── SoundBrowser.jsx   # Basic sound list
└── package.json
```

### 7.2 Feature Matrix

| Feature | Full Strudel | Minimal Strudel |
|---------|--------------|-----------------|
| Pattern library | ✅ | ✅ |
| Mini notation | ✅ | ✅ |
| Web Audio output | ✅ | ✅ |
| Samples | ✅ | ✅ |
| Synths | ✅ | ✅ |
| Effects | ✅ | ✅ |
| Music theory (scales/chords) | ✅ | ⚠️ Optional |
| MIDI output | ✅ | ❌ |
| OSC output | ✅ | ❌ |
| CSound | ✅ | ❌ |
| Hydra visuals | ✅ | ❌ |
| Soundfonts | ✅ | ❌ |
| Microtonal (xen) | ✅ | ❌ |
| Gamepad input | ✅ | ❌ |
| Motion sensors | ✅ | ❌ |
| Serial/MQTT | ✅ | ❌ |
| Haskell syntax | ✅ | ❌ |
| Desktop app | ✅ | ❌ |
| Pattern sharing | ✅ | ❌ |
| PWA/Offline | ✅ | ❌ |
| Multi-language docs | ✅ | ❌ |
| Advanced editor features | ✅ | ⚠️ Basic |
| Complex visualizations | ✅ | ⚠️ Simple |

---

## 8. Implementation Strategy

### Phase 1: Package Pruning
1. Fork to `strudel-minimal` branch
2. Remove 15+ integration/experimental packages
3. Update root package.json dependencies
4. Update website package.json to remove cut packages
5. Test that core functionality still works

### Phase 2: UI Simplification
1. Simplify settings.mjs (remove 15+ settings)
2. Simplify Header component
3. Replace multi-tab panel with single output panel
4. Remove advanced editor features
5. Create simple landing page

### Phase 3: Documentation Streamline
1. Remove multi-language support
2. Reduce to 5-10 core documentation pages
3. Remove Algolia search
4. Remove PWA features
5. Remove Supabase integration
6. Simplify config.ts

### Phase 4: Dependency Cleanup
1. Remove unused npm dependencies
2. Update package-lock files
3. Optimize bundle size
4. Test builds

### Phase 5: Testing & Polish
1. Test all core features work
2. Ensure samples and synths work
3. Verify mini notation parsing
4. Check responsive design
5. Performance testing

---

## 9. Benefits of Minimal Version

### For Users:
- **Faster loading**: Smaller bundle size (~50-60% reduction)
- **Simpler UI**: Less overwhelming for beginners
- **Focus**: Core music-making without distractions
- **Performance**: Fewer features = better performance on low-end devices
- **Clarity**: Clearer what Strudel does (live code music)

### For Developers:
- **Maintainability**: 7 packages vs 27 packages
- **Testing**: Less code to test
- **Onboarding**: Easier for new contributors
- **Build time**: Faster builds
- **Dependencies**: Fewer security vulnerabilities to monitor

### For the Project:
- **Two versions**: Full-featured vs minimal/embedded
- **Clear use cases**:
  - Full: Power users, performers, educators
  - Minimal: Beginners, embeds, workshops, mobile
- **Sustainability**: Minimal version easier to maintain long-term

---

## 10. What NOT to Cut

### Critical Components:
1. **@strudel/core** - The pattern library is the heart of Strudel
2. **@strudel/mini** - Mini notation is the primary input method
3. **@strudel/transpiler** - Needed for JavaScript syntax
4. **@strudel/webaudio** - Web Audio is the output method
5. **superdough** - The audio engine that makes sound
6. **Pattern controls** - All the core pattern manipulation functions
7. **Basic samples** - Default sound library
8. **Basic synths** - Built-in synthesizers
9. **Effects chain** - Audio effects (reverb, delay, filter, etc.)

### Nice to Have (Borderline):
1. **@strudel/tonal** - Makes musical work much easier (scales, chords)
2. **@strudel/draw** - Visual feedback is helpful for learning
3. **CodeMirror** - Could use simple textarea, but syntax highlighting is valuable
4. **Basic documentation** - At least a quick start guide

---

## 11. Estimated Size Reduction

### Package Count:
- **Before**: 27 packages
- **After**: 7 packages (5 core + 2 optional)
- **Reduction**: 74%

### Bundle Size (estimated):
- **Before**: ~5-7 MB (full website + all features)
- **After**: ~2-3 MB (minimal REPL + basic docs)
- **Reduction**: ~50-60%

### Code Lines (packages only):
- **Before**: ~15,000+ lines across packages
- **After**: ~8,000 lines (core packages only)
- **Reduction**: ~45%

### Dependencies:
- **Before**: 70+ npm dependencies (website alone)
- **After**: ~25-30 dependencies
- **Reduction**: ~60%

### Documentation Pages:
- **Before**: 70+ pages
- **After**: 10 pages
- **Reduction**: 85%

---

## Conclusion

Strudel has grown into a comprehensive live coding platform with extensive integrations and features. A minimal version focusing on core pattern-based music making could reduce complexity by ~60-70% while retaining the essential live coding experience.

**Core Philosophy for Minimal Version:**
- Make music with patterns ✅
- Use samples and synths ✅
- Write code in browser ✅
- Learn basics quickly ✅
- Everything else is optional ❌

This streamlined version would serve beginners, embedded use cases, and those who prefer simplicity, while the full version continues to serve power users and advanced performers.
