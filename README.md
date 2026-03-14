# KeyPro

**A keyboard-centric performance chart format and open-source toolset for performing musicians.**

---

## The Problem

If you are a keyboard player performing in a band, you already know this situation:

You have a chord chart — maybe from a ChordPro file, maybe from a PDF someone emailed the night before the gig, maybe handwritten on a napkin. It has lyrics and chord names. It might even have a key and a tempo. And that's about it.

Then rehearsal starts, and the singer decides the song needs to come down a whole step. You transpose in your head on the fly. You make a note in the margin. The chart now has two keys on it and you hope you remember which one is tonight's.

Now imagine you have a signature riff — the descending melodic fill you play before the chorus every time, the hook in the intro, the left-hand bass pattern under the bridge. Where does that live? In your head. On a sticky note. Nowhere the chart can see it, let alone transpose it along with everything else.

And the scale or mode? The feel? Which patch you're supposed to be on when the chorus hits? The four-bar vamp at the top before the singer comes in? None of that has a home in any chart format that exists today.

ChordPro is the closest thing the world has to a universal plain-text chart format. It is genuinely good — open, widely supported, version-control friendly, and simple enough for any text editor. But it was built by and for guitarists. Chords above lyrics, capo position, maybe a tempo. For a keyboard player, that covers perhaps half of what you actually need on stage.

KeyPro exists to cover the other half.

---

## What KeyPro Is

KeyPro is an open format specification and reference implementation for **keyboard-centric performance charts**.

It is a strict superset of ChordPro. Every valid `.chordpro` file is a valid KeyPro document. KeyPro adds a set of extension directives — all prefixed `kp-` — that give keyboard performers things no existing format provides:

### Transposable riff and phrase notation

You can write out a melodic riff — a lick, a fill, a hook, a bass line — directly in the chart, in plain note names. When you transpose the chart, the riff transposes with it. Every time.

```
{kp-riff: id="intro-hook" notes="G4 A4 B4 D5 E5" label="Intro hook" hand="right"}
```

After transposing up two semitones to A major:

```
🎹 Intro hook
 A · B · C# · E · F#
```

Define a riff once. Reference it anywhere in the chart. Change the key once. Everything moves together.

### Notation-system flexibility

Not everyone thinks in the same musical language. KeyPro supports four notation systems for riff content, and you can mix them in a single document:

- **Absolute note names** — `A C D E G` — the default, fully transposable
- **Nashville Number System** — `1 3 4 5 b7` — relative, no transposition needed
- **Western Solfege** — `Do Mi Fa Sol Ti` — relative, movable Do
- **Sargam** — `Sa Ga Ma Pa Ni` — Indian classical notation, with komal and tivra variants

If you annotate a riff in NNS or Sargam, the app has nothing to do when you change key — those systems are already relative. If you write in absolute note names, the transposition engine handles it automatically. You choose your cognitive system; KeyPro handles the rest.

### Explicit scale and mode metadata

A key signature tells you the tonic. It does not tell you whether you are in Dorian, Mixolydian, Bhairavi, or the blues scale. KeyPro makes that explicit:

```
{key: G}
{kp-scale: Mixolydian}
{kp-feel: 16th-note funk}
```

This matters for knowing which notes are available for fills, for communicating with bandmates, and for annotating Indian classical and folk material correctly.

### Semantic section markers

ChordPro has `{comment:}`. KeyPro has `{kp-section:}` — section markers that carry structural meaning, not just display text:

```
{kp-section: type="pre-chorus" label="Pre-Chorus — build here"}
{kp-section: type="bridge"}
{kp-section: type="solo" instrument="keys"}
```

In the reference application, sections appear in a navigation panel so you can jump anywhere in the song instantly during performance.

### Loose measure annotation

For performers who want bar-level structure without reading notation:

```
{kp-measure: beats=4 bars=4 label="4-bar intro vamp"}
```

Inline bar markers work exactly as they do in ChordPro:

```
[Am] | [G] | [F] [G] | [Am]
```

### Patch and MIDI hints

What sound are you supposed to be on when the chorus hits? KeyPro makes that part of the chart:

```
{kp-patch: name="Strings Pad" program=49 channel=1}
```

For split and layered keyboard setups:

```
{kp-layer: name="Right hand" range-low="C4" range-high="C7" patch="Rhodes"}
{kp-layer: name="Left hand" range-low="C2" range-high="B3" patch="Electric Bass"}
```

When MIDI output is enabled in the reference application, patch directives can optionally trigger MIDI program change messages during live performance scrolling.

---

## What KeyPro Is Not

KeyPro is not a music notation format. There are no staves, clefs, durations, or time signatures. KeyPro does not replace MusicXML, LilyPond, ABC notation, or any system designed for full musical representation.

KeyPro is for performers who do not read staff notation and have no intention of starting. If that is not you — if you read music fluently and need a notation reader — forScore, MusicXML, or MuseScore will serve you better.

KeyPro is also not a DAW or sequencer format. MIDI and patch directives are performance hints, not automation data.

---

## Who KeyPro Is For

- **Gigging keyboard players** performing in bands who work from chord charts and need riff annotations that travel with the chart when the key changes
- **Worship musicians** managing large song libraries across multiple keys, multiple services, and musicians with varying theory backgrounds
- **Session and studio keyboardists** who want to annotate their part quickly and legibly without notation software
- **Indian classical and film music performers** who think in Sargam and want a chart format that accommodates it
- **Anyone** who has ever written a transposable riff on a sticky note and lost it before the gig

---

## File Format

KeyPro files use the `.keypro` extension and are UTF-8 plain text. They are diff-friendly, edit in any text editor, and commit cleanly to Git.

A minimal KeyPro file looks like this:

```
{kp-version: 0.1}
{title: Wonderful Tonight}
{artist: Eric Clapton}
{key: G}
{kp-scale: Major}
{tempo: 96}
{kp-feel: Slow rock ballad}
{kp-patch: name="Clean Electric" program=27 channel=1}

{kp-section: type="intro"}
{kp-riff: id="intro-riff" notes="G4 F#4 E4 D4 B3" label="Intro melody line" hand="right"}

{kp-section: type="verse" label="Verse 1"}
It's [G]late in the even[D]ing,
she's [C]wondering what clothes to [D]wear

She [G]puts on her make[D]up
and [C]brushes her long blonde [D]hair

And then she [C]asks me,
[D]do I look all[G]right? [Em]

And I say [C]yes, you look [D]wonderful to[G]night [D]

{kp-section: type="chorus"}
I feel [C]wonderful because I [D]see
the love light in your [G]eyes [Em]

{kp-riff: ref="intro-riff"}
```

---

## ChordPro Compatibility

KeyPro is a strict superset of the [ChordPro specification](https://www.chordpro.org). All ChordPro directives are valid in KeyPro. All ChordPro chord notation is valid in KeyPro.

KeyPro extension directives (`kp-*`) are written in a form that conforming ChordPro parsers silently ignore as unknown directives, ensuring graceful degradation — your KeyPro files open correctly in OnSong, the ChordPro reference app, or any other ChordPro-compatible tool, with the lyrics and chords intact.

The reference implementation imports `.chordpro` files and exports back to `.chordpro`, preserving round-trip compatibility.

---

## Reference Implementation

The KeyPro reference implementation is a cross-platform application built with **.NET / C# and .NET MAUI**:

| Platform | Status |
|---|---|
| Windows | v1.0 target |
| macOS | v1.0 target |
| iOS / iPadOS | v2.0 target |
| Android | v2.0 target |

### Repository Structure

```
keypro/
├── KEYPRO_SPEC.md          ← Format specification
├── LICENSE
├── README.md               ← This file
├── src/
│   ├── KeyPro.Core/        ← Parser, AST, transposition engine (portable, no UI deps)
│   ├── KeyPro.App/         ← .NET MAUI cross-platform application
│   └── KeyPro.CLI/         ← Command-line tool (transpose, convert, validate, export)
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

### CLI Quick Start

```bash
# Transpose a file up 2 semitones
keypro transpose --semitones 2 song.keypro

# Convert a ChordPro file to KeyPro
keypro convert song.chordpro

# Validate a KeyPro file against the spec
keypro validate song.keypro

# Export to ChordPro (strips kp- directives)
keypro export --format chordpro song.keypro

# Export to PDF
keypro export --format pdf song.keypro
```

---

## Specification

The full format specification is in [`KEYPRO_SPEC.md`](./KEYPRO_SPEC.md).

It covers:

- Complete directive reference with syntax and attributes
- All four notation systems and how transposition interacts with each
- The transposition engine algorithm and enharmonic resolution rules
- Riff block definition, referencing, and multi-voice notation
- Section markers, measure annotation, and patch/MIDI directives
- ChordPro import and export rules
- Reference implementation architecture
- Full example songs across genres (Western pop/rock, funk, Hindi film, multilingual)
- Versioning and forward-compatibility rules
- Contribution guidelines and the `kp-x-` namespace for extensions

---

## Current Status

KeyPro is in active early design. The spec is at **v0.1.0 draft**. The reference implementation has not yet been released.

If you are a keyboard player, a music app developer, or someone with strong opinions about plain-text music formats, your input on the spec is welcome before v1.0 is cut.

---

## Contributing

Contributions are welcome at the spec and implementation level.

**Spec contributions** — open an issue or pull request against `KEYPRO_SPEC.md` for clarifications, new directive proposals, corrections, or additional notation system support. New directives proposed for ratification should use the `kp-x-` experimental prefix until accepted into the spec.

**Implementation contributions** — the reference implementation welcomes help with renderers, platform-specific work, MIDI device integration, and localization.

**Sample songs** — well-formed `.keypro` files across diverse genres (Western, Indian classical, film music, jazz, gospel, folk) are maintained in `samples/` as both a test corpus and a community resource. Contributions of correctly formatted samples are very welcome.

---

## License

KeyPro is released under the [MIT License](./LICENSE). The spec and reference implementation are both MIT-licensed. Go build something.

---

*Built by keyboard players, for keyboard players.*
