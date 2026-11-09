# Changelog

## r1
- First PlayStation 2 build of the ngcd core, taken from
  [NeoGeoCD-dreamcast](https://github.com/andruxaster/NeoGeoCD-dreamcast)
  r15 (raster effects, exact raster timer, wait-loop skipping, NGCD.CFG).
- 68000 on FAME C (PicoDrive's copy) with the Neo Geo's own interrupt
  vectors; Z80 on superzazu's C core.
- The core's software renderer draws into a GS texture (`NGCD_PAL_GS`: the
  palette in GS CT16 order); 240p or 480i, NTSC or PAL; pictures are
  skipped (up to 3 in a row) when the emulation falls behind.
- Sound through audsrv at 44100 Hz with the Dreamcast build's resampler
  (YM2610 at 22050 Hz + CD audio from `.NGA` files).
- Files with stdio from the ELF's folder (USB, HDD, MX4SIO, `host:`), backup
  RAM on the memory card, two DualShock pads.
- Not yet run on a PS2 or in PCSX2.
