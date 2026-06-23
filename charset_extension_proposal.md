# Charset Extension Proposal

## Objective

Allow glyph/character-set selection at runtime, independently of the emulated machine model, while preserving the current built-in behavior as a fallback.

This addresses two distinct needs:

1. Reproduce readable text for old programs.
2. Stay visually faithful to the original machine when desired.

## Current Situation

Today, the character set is effectively a bitmap compiled into the executable as a Windows resource.

Current built-in font bitmaps:

- `216_bitmap1.bmp`
- `236_bitmap1.bmp`
- `236c_bitmap1.bmp`

Selection is currently tied to machine type:

- `9816` -> `216_bitmap1.bmp`
- `9836A` -> `236_bitmap1.bmp`
- `9836C` -> `236c_bitmap1.bmp`

This selection happens at runtime, but only among resources that were embedded at build time.

Implication:

- changing a BMP on disk requires recompilation before the emulator can use it
- the machine model currently determines the font implicitly
- KML controls layout/colors/model, but not the character set directly

## Desired Direction

Decouple glyph selection from machine selection.

The emulator should be able to load a charset bitmap at runtime, so glyph behavior can be changed without rebuilding the executable.

## Proposed Runtime Model

Introduce the notion of a **charset profile**.

A charset profile selects the glyph bitmap to use for alpha text rendering, independently of the machine model.

Examples:

- `Auto`
- `9816 Original`
- `9836A Original`
- `9836C Original`
- `Roman8 Clean`
- `Mame-like`
- `Custom File`

## Recommended Architecture

### 1. Keep existing compiled resources as fallback

Do not remove the current resource-based bitmaps.

They remain the safe fallback when:

- no external charset is selected
- external loading fails
- the selected bitmap is invalid

This preserves compatibility and limits regression risk.

### 2. Add runtime loading of external BMP files

At startup or display initialization:

1. Resolve the selected charset profile.
2. If it points to an external bitmap, try to load it from disk.
3. If loading fails, fall back to the current compiled resource.

This gives flexibility without breaking the current design.

### 3. Make charset selection independent from model

Machine type should continue to define:

- display geometry
- cursor behavior
- color behavior
- other hardware-specific rendering details

But the glyph source should become a separate choice.

In other words:

- model != charset

## Solution Variants

### Option A: Direct external BMP override

Store a single path or mode in settings:

- `Auto`
- `Custom BMP path`

Behavior:

- if `Auto`, use current built-in mapping
- otherwise load the specified BMP file

Advantages:

- minimal implementation
- fast to test
- no new metadata format required

Disadvantages:

- raw file path is less user-friendly
- weak discoverability
- users can easily pick an invalid file

### Option B: Charset profiles backed by BMP files

Use named profiles instead of raw paths.

Profiles could be defined in a config file such as:

- `charsets.ini`
- or `charsets.json`

Each profile would define:

- display name
- bitmap path
- optional notes/description
- optional intended machine

Advantages:

- much cleaner user experience
- easy to ship multiple presets
- easier to document and compare

Disadvantages:

- requires config parsing
- slightly more code than Option A

### Option C: BMP base + glyph overrides

Use:

- one base BMP
- optional per-code overrides

For example:

- base from `236c`
- override `160` with HP monogram
- override `181` with triangle

Advantages:

- very flexible
- avoids duplicating large BMP files
- good for correcting a few problematic glyphs

Disadvantages:

- more complex
- requires defining an override format
- more work to validate and maintain

## Recommended First Step

Implement **Option B** with fallback behavior from **Option A**.

That means:

- named charset profiles
- each profile resolves to a BMP file
- if loading fails, fallback to built-in resources

This gives a practical and extensible structure without overengineering the first version.

## Suggested Behavior

### `Auto`

Preserve current behavior:

- `9816` -> built-in `216`
- `9836A` -> built-in `236`
- `9836C` -> built-in `236c`

### Named profiles

Allow forcing a specific charset regardless of machine:

- `9816 Original`
- `9836A Original`
- `9836C Original`
- `Roman8 Clean`
- `Mame-like`

### `Custom File`

Allow direct selection of an arbitrary BMP file.

## Technical Integration Points

The best insertion point is the font-loading logic in `display.c`, where the emulator currently chooses among:

- `IDB_216_FONTMAP1`
- `IDB_236_FONTMAP1`
- `IDB_236C_FONTMAP1`

The logic should become:

1. Determine charset profile.
2. If the profile references an external BMP, try to load it.
3. If it succeeds, use it.
4. Otherwise, load the built-in resource selected by current model logic.

## Validation Requirements

Any runtime-loaded BMP should be validated.

At minimum:

- expected width
- expected height
- compatibility with the target display mode

Potentially also:

- expected cell width
- expected cell height
- number of rows/variants in the atlas

Without validation, users may load a bitmap that breaks rendering or cursor behavior.

## Important Caveats

### 1. Different machines expect different atlas formats

The bitmaps are not interchangeable without care.

Examples:

- `216` uses a different geometry than `236`
- `236c` uses a different atlas structure and color behavior

So “independent of machine” should mean:

- user may choose another charset profile
- but the emulator should still validate whether the bitmap format is compatible

### 2. Forcing a foreign bitmap may produce visual inaccuracies

Even if a BMP loads successfully, it may not match:

- cursor placement
- underline positioning
- inverse video assumptions
- color row structure in `9836C`

This is acceptable if clearly understood as a user-controlled tradeoff.

### 3. KML should probably remain separate

KML currently controls:

- model
- layout
- colors
- background and screen placement

It does not currently control charset selection, and it is probably better to keep it that way for a first implementation.

Charset selection is better treated as:

- a user preference
- or later, a per-document preference

## Persistence Strategy

There are two natural places to store the selected charset profile:

### Global setting

Store in the Windows registry along with existing emulator settings.

Good for:

- user preference
- simplest first implementation

### Per-document setting

Store in `.E98x6` state files.

Good for:

- reproducible sessions
- preserving exact visual behavior per emulated system

Recommended order:

1. start with global setting
2. add per-document override later if needed

## Summary

The cleanest practical direction is:

- treat the charset as a selectable BMP-based runtime asset
- decouple it from machine model
- keep the current built-in resources as fallback
- introduce named charset profiles
- validate loaded BMPs before use

This approach preserves compatibility, supports original appearance, supports improved readability, and avoids requiring recompilation for glyph experiments.
