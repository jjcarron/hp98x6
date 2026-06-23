# hp98x6
This project is a fork of the HP98x6 emulator originally developed by Olivier De Smet.

The source code used as a base for this fork comes from the public repository published by calmsacibis995, which contains the GPL‑3.0 license and the original sources.

The original distribution by Olivier De Smet is also available under his site [hp98x6](https://sites.google.com/site/olivier2smet2/hp_projects/hp98x6).

Current release: `2026.06.01`

This fork preserves the original emulator sources and adds maintenance updates to keep the project usable on current systems.

## Release 2026.06.01
- Updated the 9816, 9836A, and 9836C built-in font tables for improved extended-character rendering.
- Rebuilt the 9816 extended glyph sheet from an edited review sheet.
- Kept build artifacts and local editing assets out of Git.

## Font adaptations
- The built-in alpha font bitmaps are selected by machine model at runtime:
  - `216_bitmap1.bmp` for HP 9816
  - `236_bitmap1.bmp` for HP 9836A
  - `236c_bitmap1.bmp` for HP 9836C
- The `168..222` glyph range for the 9816 and 9836A tables has been rebuilt from the 9836C table to improve consistency of extended characters.
- Glyph `223` (`0xDF`) has been set to the HP monogram in all three tables.
- On the 9836C path, some unsupported characters entered from BASIC are normalized by the original display chain before final rendering; mapping `0xDF` to the HP monogram preserves the observed machine behavior without patching ROM or BASIC code.

## TODO list
- Update the UI for modern versions of Windows
- Fix broken menu interface elements
- Port to Linux/macOS (maybe)
- An updated interface
- Some other things

Feel free to make a pull request in case you found a bug and fixed it, or that you want to add new feature to the emulator.
