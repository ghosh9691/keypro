# KeyPro Format Specification

**Version:** 0.1.0 (Draft)  
**Status:** Draft for Review  
**Authors:** KeyPro Contributors  
**License:** MIT  

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Design Philosophy](#2-design-philosophy)
3. [Relationship to ChordPro](#3-relationship-to-chordpro)
4. [File Format](#4-file-format)
5. [Metadata Directives](#5-metadata-directives)
6. [Chord Notation](#6-chord-notation)
7. [Riff Blocks](#7-riff-blocks)
8. [Notation Systems](#8-notation-systems)
9. [Section Directives](#9-section-directives)
10. [Measure Markers](#10-measure-markers)
11. [Patch and MIDI Directives](#11-patch-and-midi-directives)
12. [Transposition Engine](#12-transposition-engine)
13. [Rendering](#13-rendering)
14. [Conversion from ChordPro](#14-conversion-from-chordpro)
15. [Reference Implementation](#15-reference-implementation)
16. [Example Songs](#16-example-songs)
17. [Versioning and Compatibility](#17-versioning-and-compatibility)
18. [Contributing](#18-contributing)

---

## 1. Introduction

KeyPro is an open format specification for keyboard-centric performance charts. It is a superset of the ChordPro format, extending it with directives specifically designed for keyboard players who:

- Perform from lyrics-and-chords charts rather than staff notation
- Need to annotate melodic riffs and phrases in a transposable way
- Work in different notation systems (absolute note names, Nashville Number System, Solfege/Sargam)
- Require patch/sound metadata alongside chart content
- Perform in bands where real-time transposition is routine

KeyPro files use the `.keypro` extension. A conforming KeyPro parser **must** also parse valid `.chordpro` files. A `.chordpro` file with no KeyPro-specific directives is a valid KeyPro document.

---

## 2. Design Philosophy

### 2.1 Principles

**Non-readers first.** KeyPro is designed for keyboard performers who do not read staff notation and have no intention of doing so. There are no staves, clefs, or time signatures in the rendered output. Notes are expressed as names, numbers, or syllables — whichever the performer thinks in.

**Notation-system agnostic.** A performer may annotate riffs using absolute note names (`A C D E G`), Nashville Number System (`1 3 4 5 b7`), Western Solfege (`Do Mi Fa Sol Ti`), or Indian Sargam (`Sa Ga Ma Pa Ni`). The format accommodates all four. The transposition engine acts only on absolute note names; relative systems are stored verbatim and displayed as authored.

**Transposition as a first-class operation.** Every element of a KeyPro document that expresses pitch in absolute terms — chords, riff note sequences, key metadata — participates in transposition. Nothing is left behind when the key changes.

**ChordPro compatibility.** Any valid `.chordpro` file opens correctly in a KeyPro application. KeyPro directives are written in a form that ChordPro parsers silently ignore as unknown directives, ensuring graceful degradation.

**Reuse over repetition.** Riff blocks can be defined once and referenced many times within a song. A chorus riff defined at the top of the file need not be rewritten for each chorus occurrence.

**Plain text, version-control friendly.** KeyPro files are UTF-8 plain text. They diff cleanly, can be edited in any text editor, and commit well to Git.

### 2.2 Explicit Non-Goals

- KeyPro is **not** a music notation format. It does not encode rhythm, duration, dynamics, or articulation.
- KeyPro is **not** a replacement for MusicXML, LilyPond, or ABC notation for purposes requiring full musical representation.
- KeyPro is **not** a DAW integration format. MIDI and patch directives are hints for live performance context, not sequencer automation data.
- KeyPro does **not** target classical musicians who read staff notation. forScore and similar tools serve that audience well.

---

## 3. Relationship to ChordPro

### 3.1 ChordPro Compatibility

KeyPro is a strict superset of the ChordPro specification (as defined at [chordpro.org](https://www.chordpro.org)). All ChordPro directives are valid in KeyPro. All ChordPro chord notation (`[Am]`, `[G/B]`, etc.) is valid in KeyPro.

A KeyPro parser must:
- Parse and render all standard ChordPro directives
- Parse and act on KeyPro extension directives (prefixed `kp-` or using `{keypro_*}` namespace)
- Silently ignore unknown directives (ChordPro-standard behavior)

### 3.2 ChordPro Directives Superseded by KeyPro

The following ChordPro directives have KeyPro equivalents that carry additional semantics. Both forms remain valid; the KeyPro form is preferred in `.keypro` files:

| ChordPro | KeyPro Equivalent | Additional semantics |
|---|---|---|
| `{title:}` | `{title:}` | Unchanged |
| `{key:}` | `{key:}` + `{kp-scale:}` | Scale/mode made explicit |
| `{comment:}` | `{kp-section:}` | Semantic section type |
| `{tempo:}` | `{tempo:}` | Unchanged |

### 3.3 Conversion

A KeyPro application must provide import of `.chordpro` files, producing `.keypro` files with:
- All existing directives preserved verbatim
- `{comment:}` directives optionally promoted to `{kp-section:}` by the user
- A prompt to add `{kp-scale:}` and `{kp-patch:}` metadata

Export back to `.chordpro` must strip all `kp-` prefixed directives and render riff blocks as `{comment:}` blocks.

---

## 4. File Format

### 4.1 Encoding

KeyPro files are UTF-8 encoded plain text. Line endings may be LF or CRLF. A conforming parser must accept both.

### 4.2 File Extension

- `.keypro` — KeyPro native format
- `.chordpro`, `.cho`, `.crd` — Accepted for import; must be treated as valid KeyPro input

### 4.3 Line Structure

A KeyPro document consists of lines, each of which is one of:

- **Directive line:** begins with `{` and ends with `}`, e.g. `{title: Blackbird}`
- **Lyric/chord line:** contains lyrics with optional inline chord markers in `[brackets]`
- **Riff definition line:** a `{kp-riff:}` directive block (may span logical lines via continuation)
- **Empty line:** separates verses/sections
- **Comment line:** begins with `#`, ignored by parser

### 4.4 Directive Syntax

```
{directive-name: value}
{directive-name: key1="value1" key2="value2"}
```

KeyPro extension directives use the `kp-` prefix:

```
{kp-scale: Dorian}
{kp-riff: id="intro" notes="G A B D E" label="Intro hook"}
```

---

## 5. Metadata Directives

All metadata directives should appear at the top of the file, before any lyric content.

### 5.1 Core Metadata

```
{title: Song Title}
{subtitle: Optional subtitle or artist name}
{artist: Artist Name}
{album: Album Name}
{year: 1975}
{key: G}
{tempo: 120}
{time: 4/4}
```

### 5.2 KeyPro Metadata Extensions

#### `{kp-scale:}` — Mode/Scale annotation

Specifies the scale or mode of the song, independent of key. This is display metadata; it does not affect transposition.

```
{kp-scale: Major}
{kp-scale: Dorian}
{kp-scale: Mixolydian}
{kp-scale: Minor}
{kp-scale: Pentatonic Minor}
{kp-scale: Blues}
{kp-scale: Bhairavi}      ← Indian raga name accepted
{kp-scale: Kafi}
```

Scale names are freeform strings. The reference implementation ships with a lookup table of common Western modes, pentatonic scales, blues scales, and common Indian ragas for display enrichment (e.g., showing scale degrees on hover).

#### `{kp-feel:}` — Rhythmic feel annotation

A freeform string describing groove or feel. Display only.

```
{kp-feel: Slow ballad}
{kp-feel: Uptempo swing}
{kp-feel: 16th-note funk}
{kp-feel: Bhangra}
```

#### `{kp-capo:}` — Effective capo for band context

When a guitarist in the band uses a capo, chord names in the chart may need to reflect that context. This is an informational annotation.

```
{kp-capo: 2}
```

#### `{kp-notation:}` — Default notation system for riffs

Sets the default notation system for riff blocks in this document. Can be overridden per riff block.

```
{kp-notation: absolute}    ← default; note names like A, Bb, C#
{kp-notation: nns}         ← Nashville Number System
{kp-notation: solfege}     ← Western: Do Re Mi Fa Sol La Ti
{kp-notation: sargam}      ← Indian: Sa Re Ga Ma Pa Dha Ni
```

#### `{kp-version:}` — Spec version

Declares the KeyPro spec version this file targets.

```
{kp-version: 0.1}
```

---

## 6. Chord Notation

KeyPro inherits ChordPro chord notation unchanged. Chords appear inline within lyric lines, enclosed in square brackets.

```
[Am]Here comes the [G]sun, [F]little darling
```

### 6.1 Supported Chord Symbols

- Simple triads: `C`, `Dm`, `Em`, `F`, `G`, `Am`, `Bdim`
- Seventh chords: `Cmaj7`, `G7`, `Am7`, `Dm7b5`
- Extensions: `C9`, `G13`, `Fsus4`, `Dsus2`
- Slash chords: `G/B`, `C/E`, `Am/G`
- Altered: `G7#5`, `Bb7b9`

### 6.2 Enharmonic Preference

The transposition engine respects enharmonic preference. Songs using flat keys (`Bb`, `Eb`, `Ab`, `Db`, `Gb`) will transpose into flat-preferring keys. Songs using sharp keys (`G`, `D`, `A`, `E`, `B`, `F#`) will transpose into sharp-preferring keys.

A `{kp-enharmonic: flats}` or `{kp-enharmonic: sharps}` directive overrides this behavior globally.

---

## 7. Riff Blocks

Riff blocks are the primary KeyPro extension. They allow a keyboard performer to notate a melodic phrase — a riff, lick, fill, or hook — in plain-text note names, embedded in context within the chart.

### 7.1 Defining a Riff

```
{kp-riff: id="intro-hook" notes="G A B D E" label="Intro hook" octave="4"}
```

**Attributes:**

| Attribute | Required | Description |
|---|---|---|
| `id` | Yes (if referenced elsewhere) | Unique identifier within the document |
| `notes` | Yes | Space-separated note sequence (see §8) |
| `label` | No | Human-readable display label |
| `octave` | No | Default octave for notes without explicit octave (default: 4) |
| `notation` | No | Overrides `{kp-notation:}` for this riff |
| `hand` | No | `left`, `right`, or `both` — performance hint |
| `comment` | No | Freeform annotation text |

### 7.2 Referencing a Riff

A defined riff can be placed anywhere in the chart by reference:

```
{kp-riff: ref="intro-hook"}
```

When the transposition engine runs, all instances — definition and references — display the transposed version. The definition is the single source of truth.

### 7.3 Inline Riff (Anonymous)

A riff that appears only once need not have an `id`:

```
{kp-riff: notes="E4 D4 B3 G3" label="Pre-chorus descending fill"}
```

### 7.4 Riff Rendering

In the rendered chart, a riff block appears as a visually distinct inline block, clearly differentiated from lyric lines. The label appears as a header; notes appear as a horizontal sequence of note pills or tokens. Example rendering:

```
╔══════════════════════════════════╗
║ 🎹 Intro hook                    ║
║  G · A · B · D · E               ║
╚══════════════════════════════════╝
```

After transposition to A major (+2 semitones):

```
╔══════════════════════════════════╗
║ 🎹 Intro hook                    ║
║  A · B · C# · E · F#             ║
╚══════════════════════════════════╝
```

### 7.5 Multi-voice Riffs

For riffs that specify both hands:

```
{kp-riff: id="chorus-stab"
  right="E4 D4 B3"
  left="G2 D3"
  label="Chorus stab chord"
  comment="Hold left hand, step right hand down"}
```

---

## 8. Notation Systems

KeyPro supports four notation systems for riff note content. The system in use is declared per-document via `{kp-notation:}` and may be overridden per riff block.

### 8.1 Absolute Note Names (default)

Notes expressed as pitch class names with optional octave and accidentals.

**Format:** `[A-G][b|#]?[octave]?`

```
notes="A C D E G"            ← pitch class only, no octave
notes="A3 C4 D4 E4 G4 A4"   ← with Scientific Pitch Notation octave
notes="Bb3 D4 F4"            ← flat accidentals
notes="F#4 A4 C#5"           ← sharp accidentals
```

**Transposition:** The transposition engine applies the semitone offset to each note, preserving octave relationships. Enharmonic spelling follows the key's preference (see §6.2).

**Octave resolution:** When octave numbers are omitted, the `octave` attribute of the riff block sets the default. Notes are assumed to be in ascending or stepwise motion unless octave numbers indicate otherwise.

### 8.2 Nashville Number System (NNS)

Notes expressed as scale degrees relative to the tonic.

**Format:** `[b|#]?[1-7]`

```
notes="1 3 4 5 b7"
notes="1 2 3 5 6"
notes="b3 4 5 b7 8"     ← 8 = octave above root
```

**Transposition:** No transformation applied. NNS is inherently relative. The parser stores the sequence verbatim. The renderer displays it with the key context visible.

### 8.3 Western Solfege

Notes expressed as movable-do solfege syllables.

**Format:** `[Do|Re|Mi|Fa|Sol|La|Ti]` with optional `b` (flat) prefix

```
notes="Do Mi Fa Sol Ti"
notes="Do Re Mi Sol La Do"
notes="bMi Fa Sol bTi Do"
```

**Transposition:** No transformation applied. Solfege is inherently relative (movable Do).

### 8.4 Sargam (Indian Classical)

Notes expressed as Indian sargam syllables, supporting komal (flat) and tivra (sharp) variants.

**Format:** `[Sa|Re|Ga|Ma|Pa|Dha|Ni]` with optional `k` (komal) or `t` (tivra) suffix, or lowercase for komal convention.

```
notes="Sa Re Ga Ma Pa Dha Ni"    ← Bilawal (natural)
notes="Sa Re ga Ma Pa Dha ni"    ← lowercase = komal
notes="Sa Re ga Ma Pa dha ni"    ← Kafi thaat
notes="Sa Re Ga Ma(t) Pa Dha Ni" ← Ma tivra (Kalyan thaat)
```

**Transposition:** No transformation applied. Sargam is inherently relative (Sa = tonic).

### 8.5 Mixed Notation

A single document may contain riff blocks in different notation systems. Each riff block declares its own system:

```
{kp-notation: absolute}   ← document default

{kp-riff: id="verse-lick" notes="A3 C4 D4 E4" label="Verse lick"}
{kp-riff: id="chorus-feel" notes="1 3 4 5" notation="nns" label="Chorus feel (NNS)"}
{kp-riff: id="bridge-phrase" notes="Sa Re Ga Ma" notation="sargam" label="Bridge phrase"}
```

---

## 9. Section Directives

KeyPro extends ChordPro's `{comment:}` with semantic section markers that carry structural meaning.

### 9.1 `{kp-section:}` — Named section with type

```
{kp-section: type="verse" label="Verse 1"}
{kp-section: type="chorus" label="Chorus"}
{kp-section: type="bridge"}
{kp-section: type="intro"}
{kp-section: type="outro"}
{kp-section: type="solo" instrument="keys"}
{kp-section: type="interlude"}
{kp-section: type="pre-chorus"}
{kp-section: type="tag"}
```

**`type` values:** `intro`, `verse`, `pre-chorus`, `chorus`, `post-chorus`, `bridge`, `solo`, `interlude`, `outro`, `tag`, `instrumental`, `break`, `custom`

For `type="custom"`, the `label` attribute is used as the display name.

### 9.2 Section Navigation

In the reference implementation, declared sections appear in a navigation sidebar or jump menu, allowing the performer to tap/click to any section during performance.

### 9.3 Compatibility

`{kp-section:}` renders as a visually prominent section header. When exporting to `.chordpro`, it is emitted as `{comment: label}`.

---

## 10. Measure Markers

KeyPro provides loose, non-strict measure annotation. This is not a time signature enforcement mechanism — it is a visual and cognitive aid for performers who want to track bar structure without reading notation.

### 10.1 Inline Measure Bar

The pipe character `|` on a lyric/chord line indicates a bar line, inherited from ChordPro convention:

```
[Am] | [G] | [F] [G] | [Am]
```

### 10.2 `{kp-measure:}` — Annotated measure block

```
{kp-measure: beats=4 label="4-bar intro vamp" bars=4}
{kp-measure: beats=3 label="Waltz feel section"}
{kp-measure: beats=4 feel="half-time" bars=8 label="Breakdown"}
```

**Attributes:**

| Attribute | Description |
|---|---|
| `beats` | Beats per bar (display hint, not enforced) |
| `bars` | Number of bars this block covers |
| `label` | Freeform description |
| `feel` | Rhythmic feel override for this block |

### 10.3 Rendering

Measure annotations appear as subtle block annotations above or alongside lyric lines — not interrupting the reading flow, but visible at a glance for orientation.

---

## 11. Patch and MIDI Directives

KeyPro carries performance context — specifically, what sound the keyboard should be producing at any point in the chart. These are hints, not commands. The reference implementation can optionally send MIDI program change messages when a patch directive is encountered during live performance scrolling.

### 11.1 `{kp-patch:}` — Sound/patch annotation

```
{kp-patch: name="Grand Piano" channel=1 program=0 bank=0}
{kp-patch: name="Strings Pad" channel=1 program=49}
{kp-patch: name="Rhodes" channel=1 program=4}
{kp-patch: name="Organ" channel=2 program=16}
```

**Attributes:**

| Attribute | Required | Description |
|---|---|---|
| `name` | Yes | Human-readable patch name (display) |
| `channel` | No | MIDI channel (1–16) |
| `program` | No | MIDI program number (0–127) |
| `bank` | No | MIDI bank number |

`{kp-patch:}` placed at document level sets the default patch for the song. `{kp-patch:}` placed inline within the lyric flow indicates a patch change at that point.

### 11.2 `{kp-layer:}` — Split/layer annotation

For performers using keyboard splits or layers (e.g., left hand on bass, right hand on piano):

```
{kp-layer: name="Left hand" range-low="C2" range-high="B3" patch="Electric Bass"}
{kp-layer: name="Right hand" range-low="C4" range-high="C7" patch="Grand Piano"}
```

This is informational metadata for the performer. The reference implementation may render it as a keyboard diagram.

### 11.3 `{kp-tempo-change:}` — Inline tempo change

```
{kp-tempo-change: bpm=96 label="Double time feel"}
{kp-tempo-change: bpm=72 label="Back to half time"}
```

---

## 12. Transposition Engine

### 12.1 What Transposes

When a transposition operation is applied (semitone offset `n`, positive = up, negative = down), the following elements are transformed:

- `{key:}` metadata value
- All inline chord symbols `[ChordName]`
- All `{kp-riff:}` note sequences where `notation="absolute"` (or default)
- `{kp-patch:}` directives are **not** transposed (patch names are instrument-independent)
- NNS, Solfege, and Sargam riff sequences are **not** transposed (they are relative)

### 12.2 Transposition Algorithm

For each absolute pitch (chord root or riff note):

1. Map the pitch name to a chromatic index (C=0, C#/Db=1, D=2, ... B=11)
2. Add the semitone offset `n`, modulo 12
3. Map back to a pitch name, using the enharmonic preference of the target key
4. Preserve octave number if present; adjust for octave boundaries (e.g., B4 + 2 = C#5)

### 12.3 Enharmonic Resolution

Target key enharmonic preference:

| Key | Preference |
|---|---|
| C, G, D, A, E, B, F#, C# | Sharps |
| F, Bb, Eb, Ab, Db, Gb, Cb | Flats |

### 12.4 Transposition Scope

Transposition operates at the document level and is non-destructive. The application maintains the original document and applies transposition as a view-layer transform. The user may:

- Apply transposition for the current session without modifying the file
- Save a transposed version as a new document
- Store multiple named transpositions per song (e.g., "Original — G", "Singer's key — F", "Capo 2 context — A")

---

## 13. Rendering

### 13.1 Display Modes

A KeyPro reference implementation should support at minimum:

**Performance view** — Large, high-contrast, auto-scrollable. Optimized for reading while playing. Chords prominently displayed inline with lyrics. Riff blocks visually distinct. Section markers prominent.

**Edit view** — The raw `.keypro` text, with syntax highlighting. Direct editing with live preview.

**Print/export view** — Clean PDF export, paginated, with riff blocks rendered compactly.

### 13.2 Chord Display Options

- Inline with lyrics (default, and KeyPro-preferred)
- Above lyrics (ChordPro-standard, supported for compatibility)

### 13.3 Riff Block Rendering

Riff blocks render as visually enclosed blocks distinct from the lyric flow. The note sequence is displayed horizontally with clear separators. The label appears as a title. Hand annotations (`left`, `right`, `both`) render as a subtle indicator.

For absolute notation, the current transposition is applied before display. For relative notations (NNS, Solfege, Sargam), the sequence is shown as-is, with the current key shown in context.

### 13.4 Theming

The reference implementation supports light and dark themes. A high-contrast "stage mode" theme optimizes for dark stage environments.

---

## 14. Conversion from ChordPro

A conforming KeyPro application must support importing `.chordpro` files via the following process:

1. **Parse** the ChordPro file fully, preserving all directives and content
2. **Promote** `{comment:}` directives to `{kp-section:}` with `type="custom"`, prompted by user
3. **Add** `{kp-version: 0.1}` at top of file
4. **Preserve** all original ChordPro content verbatim
5. **Prompt** the user to optionally add `{kp-scale:}`, `{kp-feel:}`, and `{kp-patch:}` metadata

The resulting file is saved as `.keypro`. The original `.chordpro` file is preserved unless the user explicitly overwrites it.

**Export to ChordPro:**

1. Strip all `kp-` prefixed directives
2. Convert `{kp-section:}` to `{comment: label}`
3. Convert `{kp-riff:}` blocks to `{comment: label: notes}` (display-only)
4. Output as `.chordpro`

---

## 15. Reference Implementation

### 15.1 Technology

The reference implementation is a cross-platform desktop application built with **.NET / C# and .NET MAUI**, targeting:

- **Windows** (primary, v1.0)
- **macOS** (primary, v1.0)
- **iOS/iPadOS** (v2.0)
- **Android** (v2.0)

### 15.2 Repository Structure

```
keypro/
├── KEYPRO_SPEC.md          ← This document
├── LICENSE
├── README.md
├── src/
│   ├── KeyPro.Core/        ← Parser, AST, transposition engine (portable)
│   ├── KeyPro.App/         ← .NET MAUI cross-platform application
│   └── KeyPro.CLI/         ← Command-line tool (transpose, convert, validate)
├── tests/
│   ├── KeyPro.Core.Tests/
│   └── KeyPro.App.Tests/
├── samples/
│   ├── sample-basic.chordpro
│   ├── sample-full.keypro
│   └── sample-multilingual.keypro
└── tools/
    └── chordpro-import/    ← Batch ChordPro → KeyPro converter
```

### 15.3 Core Library (KeyPro.Core)

The core library is framework-independent and contains:

- **Parser:** Tokenizer and parser producing a `SongDocument` AST
- **AST types:** `SongDocument`, `SongLine`, `LyricLine`, `DirectiveLine`, `RiffBlock`, `SectionMarker`, `MeasureBlock`, `InlineChord`, `RiffNote`
- **Transposition engine:** Pure function `Transpose(SongDocument doc, int semitones) → SongDocument`
- **Renderer interfaces:** `IChartRenderer` — implemented by the app layer for screen and print
- **ChordPro importer:** `ChordProImporter.Import(string chordproText) → SongDocument`
- **ChordPro exporter:** `ChordProExporter.Export(SongDocument doc) → string`

### 15.4 CLI Tool

```bash
# Transpose a file up 2 semitones
keypro transpose --semitones 2 song.keypro

# Convert ChordPro to KeyPro
keypro convert song.chordpro

# Validate a KeyPro file
keypro validate song.keypro

# Export to ChordPro
keypro export --format chordpro song.keypro

# Export to PDF
keypro export --format pdf song.keypro
```

### 15.5 MIDI Integration (Optional Module)

When MIDI output is enabled in the application, `{kp-patch:}` directives encountered during scrolled performance playback trigger MIDI Program Change messages on the configured output device. This is an optional feature that does not affect core parsing or rendering.

---

## 16. Example Songs

### 16.1 Minimal KeyPro file (converted from ChordPro)

```
{kp-version: 0.1}
{title: Wonderful Tonight}
{artist: Eric Clapton}
{key: G}
{kp-scale: Major}
{tempo: 96}
{kp-feel: Slow rock ballad}
{kp-patch: name="Clean Electric" program=27 channel=1}

{kp-section: type="intro" label="Intro"}
{kp-riff: id="intro-riff" notes="G4 F#4 E4 D4 B3" label="Intro melody line" hand="right"}

{kp-section: type="verse" label="Verse 1"}
It's [G]late in the even[D]ing,
she's [C]wondering what clothes to [D]wear

She [G]puts on her make[D]up
and [C]brushes her long blonde [D]hair

And then she [C]asks me,
[D]do I look all[G]right? [Em]

And I say [C]yes, you look [D]wonderful to[G]night [D]

{kp-section: type="chorus" label="Chorus"}
I feel [C]wonderful because I [D]see
the love light in your [G]eyes [Em]

And the [C]wonder of it [D]all
is that you just don't real[G]ize
how much I [D]love [G]you

{kp-riff: ref="intro-riff"}
```

---

### 16.2 Full KeyPro file with riffs, NNS, and patch changes

```
{kp-version: 0.1}
{title: Sample Funk Groove}
{key: E}
{kp-scale: Mixolydian}
{tempo: 108}
{kp-feel: 16th-note funk}
{kp-notation: absolute}
{kp-patch: name="Rhodes" program=4 channel=1}

{kp-layer: name="Right hand" range-low="C4" range-high="C7" patch="Rhodes"}
{kp-layer: name="Left hand" range-low="C2" range-high="B3" patch="Electric Bass"}

{kp-section: type="intro" label="Intro — 4 bar vamp"}
{kp-measure: beats=4 bars=4 label="Groove vamp — no chord changes"}
{kp-riff: id="main-groove" notes="E4 G4 A4 B4 D5 B4 A4 G4" label="Main right-hand hook" hand="right"}
{kp-riff: id="bass-line" notes="E2 E2 G2 A2" label="Bass left hand" hand="left" notation="absolute"}

[E7]  |  [E7]  |  [A9]  [E7]  |  [B7#9]  [A9]  [E7]

{kp-section: type="verse" label="Verse"}
[E7]Come on now [A9]feel the groove
[E7]Lay it [B7#9]down and [A9]let it [E7]move

{kp-section: type="pre-chorus" label="Pre-Chorus"}
{kp-riff: id="pre-ch-fill" notes="D5 C#5 B4 A4 G#4 F#4 E4" label="Pre-chorus descending fill" hand="right"}
[A]Build it [B]up now

{kp-section: type="chorus" label="Chorus"}
{kp-patch: name="Organ" program=16 channel=1}
[E7]Let the [A9]music [E7]take you
{kp-riff: ref="main-groove"}

{kp-section: type="bridge" label="Bridge — NNS reference"}
{kp-riff: id="bridge-motif" notes="1 b3 4 5 b7" notation="nns" label="Bridge motif (relative)" comment="Play sparse, lots of space"}

{kp-patch: name="Rhodes" program=4 channel=1}
{kp-section: type="outro" label="Outro — fade"}
{kp-riff: ref="main-groove"}
[E7]  |  [E7]  |  [A9]  [E7]  |  [E7]
```

---

### 16.3 Hindi film song example with Sargam annotation

```
{kp-version: 0.1}
{title: Tujhe Dekha To}
{artist: Lata Mangeshkar / Kumar Sanu}
{album: Dilwale Dulhania Le Jayenge}
{year: 1995}
{key: G}
{kp-scale: Kafi}
{tempo: 72}
{kp-feel: Slow romantic ballad}
{kp-notation: sargam}
{kp-patch: name="Strings Pad" program=49 channel=1}

{kp-section: type="intro" label="Intro"}
{kp-riff: id="intro-sargam" notes="Sa Re Ga Ma Pa Dha Ni Sa" notation="sargam" label="Intro ascending phrase" hand="right"}

{kp-section: type="verse" label="Mukhda"}
[G]Tujhe dekha to [D]ye jaana sanam
[Em]Pyaar hota hai [C]deewana sanam
[G]Abhi jaana [D]sanam
[C]Hota hai [D]pyaar [G]tera

{kp-riff: id="antara-fill" notes="Ni Dha Pa Ma Ga Re Sa" notation="sargam" label="Antara descending fill" hand="right"}

{kp-section: type="chorus" label="Antara"}
{kp-patch: name="Flute" program=73 channel=2}
[Em]Mere sapnon ki rani [C]kab aayegi tu
[G]Aaya mausam aawara[D]pun ka
{kp-riff: ref="antara-fill"}

{kp-patch: name="Strings Pad" program=49 channel=1}
{kp-section: type="interlude" label="Interlude"}
{kp-riff: ref="intro-sargam"}
```

---

## 17. Versioning and Compatibility

### 17.1 Spec Versioning

KeyPro uses Semantic Versioning for the spec itself:

- **Major version** bump: breaking changes to directive syntax or transposition semantics
- **Minor version** bump: new directives or attributes added (backward compatible)
- **Patch version** bump: clarifications, corrections to the spec text

### 17.2 Forward Compatibility

A parser encountering a `{kp-version:}` higher than its supported version must:
- Parse and render all directives it recognizes
- Silently ignore directives it does not recognize
- Display a non-fatal warning that the file targets a newer spec version

### 17.3 Application Versioning

Application version and spec version are independent. An application may support multiple spec versions simultaneously.

---

## 18. Contributing

### 18.1 Spec Contributions

Issues and pull requests against `KEYPRO_SPEC.md` are welcome for:
- Clarifications to ambiguous language
- New directive proposals (must include use case, syntax, and example)
- Corrections to examples
- Additional notation system support

New directives proposed for inclusion in the spec should be prefixed `kp-x-` (experimental) until ratified.

### 18.2 Implementation Contributions

The reference implementation welcomes contributions in:
- Additional renderers (HTML, SVG, etc.)
- Platform-specific improvements
- MIDI device integration
- Notation system support
- Localization

### 18.3 Song Library

A `samples/` directory of well-formed `.keypro` files representing diverse genres (Western pop/rock, Hindi film, Bengali folk, worship, jazz, blues) is maintained as a test corpus and community resource.

---

## Appendix A — Chromatic Index Reference

| Note | Index |
|---|---|
| C | 0 |
| C# / Db | 1 |
| D | 2 |
| D# / Eb | 3 |
| E | 4 |
| F | 5 |
| F# / Gb | 6 |
| G | 7 |
| G# / Ab | 8 |
| A | 9 |
| A# / Bb | 10 |
| B | 11 |

---

## Appendix B — Supported Scale Names (Reference)

Western modes: `Major`, `Minor`, `Dorian`, `Phrygian`, `Lydian`, `Mixolydian`, `Locrian`, `Harmonic Minor`, `Melodic Minor`

Pentatonic/Blues: `Pentatonic Major`, `Pentatonic Minor`, `Blues`

Indian thaats (common): `Bilawal`, `Kafi`, `Khamaj`, `Bhairav`, `Bhairavi`, `Kalyan`, `Marwa`, `Purvi`, `Todi`, `Asavari`

Freeform strings are accepted for any other scale or raga name.

---

## Appendix C — Reserved `kp-` Directive Names

The following directive names are reserved for future use and must not be used by third-party extensions:

`kp-chord`, `kp-lyric`, `kp-verse`, `kp-chorus`, `kp-bridge`, `kp-key`, `kp-time`, `kp-repeat`, `kp-coda`, `kp-segno`, `kp-fine`

Third-party extension directives should use a namespaced prefix: `kp-x-vendorname-directivename`.

---

*KeyPro Specification v0.1.0 — Draft*  
*This document is released under the MIT License.*  
*Contributions welcome at: [github.com/keypro/keypro-spec] (placeholder)*
