# hp98x6
A fork of Olivier De Smet's HP98x6 emulator.

This emulator is in really rough shape. Some menu interfaces don't work right, a broken Windows XP theme, and a whole bunch of other issues. What's worse,
is that the emulator isn't available for download any more. I used the Wayback Machine to download the source code.

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
