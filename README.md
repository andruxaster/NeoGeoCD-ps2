# ngcd-ps2 — Neo Geo CD emulator for the PlayStation 2

A low-level Neo Geo CD (CDZ) emulator for the PlayStation 2, built with
ps2sdk and gsKit. It runs the real CDZ BIOS and unmodified games: the
LC8951 CD-ROM decoder, the CD drive protocol, DMA, the LSPC video chip, the
YM2610 and the sound Z80 are emulated. The core is the one of
[NeoGeoCD-dreamcast](https://github.com/andruxaster/NeoGeoCD-dreamcast)
(its behaviour follows NeoCD / neocd_libretro by Fabrice Martinez,
rewritten in C); this repository adds C CPU cores and a PS2 front end.

BIOS files and games are not included.

## Status

**Trial build: not yet run on a PS2 or in PCSX2.** It builds and links with
the current ps2dev toolchain (GCC 15.2). The core itself is the tested one
of the Dreamcast build; what is new here (FAME C, video, sound, pads and
file access on the PS2) still has to be tried on the console.

Speed, measured without a console (instructions counted under
qemu-mipsel, Metal Slug, Samurai Shodown IV and Neo Turf Masters):

| | MIPS instructions a frame (average) |
|---|---|
| emulation, 68000 on FAME C | 2.2–2.6 M |
| emulation, 68000 on Musashi (for comparison) | 3.7–4.7 M |
| software renderer | 1.1–3 M |

How that turns into milliseconds on the EE (294.9 MHz) is not known yet.
When the emulation falls behind the TV, the picture is skipped (up to 3
frames in a row) and sound keeps going. The debug build prints, every 300
frames, how many frames were shown (see [Logs](#logs)).

## Building

Requires ps2dev (ps2sdk, gsKit and ps2_drivers) with its environment set
(`PS2DEV`, `PS2SDK`; `GSKIT` defaults to `$PS2DEV/gsKit`).

    make              # ngcd-ps2.elf (stripped) and ngcd.elf, with a console log
    make RELEASE=1    # without the log
    make clean

## Game files

A Neo Geo CD image (CUE + BIN: a MODE1/2352 data track and CD-DA tracks)
is converted by `tools/ngcdconv` (the same files as for the Dreamcast
build):

    cc -O2 -o ngcdconv tools/ngcdconv/ngcdconv.c
    ./ngcdconv "game.cue" out

| File | From | Size |
|---|---|---|
| `NEOCD.BIN` | CDZ BIOS `neocd.bin` (either byte order) | 512 KB |
| `NG_LO.ROM` | `000-lo.lo` / `ng-lo.rom`; optional, generated if missing | 64 KB |
| `TOC.BIN` | `ngcdconv`: track and index table | |
| `DATA.ISO` | `ngcdconv`: the data track, 2048-byte sectors | |
| `TRKnn.NGA` | `ngcdconv`: CD-DA tracks as IMA ADPCM 44.1 kHz stereo | |
| `NGCD.CFG` | optional settings | |

The files go into one folder together with the ELF; the emulator reads
them from the folder it was started from, or from the folder given as the
ELF's first argument, or from `mass:/NGCD/` when it gets no path at all.
One game per folder (copy the ELF into each).

## Running on a console with FreeMcBoot

1. Format a USB stick as FAT32 with an MBR partition table (sticks of up
   to 32 GB are the safest).
2. Put the ELF and the game's files in a folder, for example:

       mass:/NGCD/
           ngcd-ps2.elf
           NEOCD.BIN  NG_LO.ROM  TOC.BIN  DATA.ISO  TRK02.NGA ...
           NGCD.CFG             (optional)

3. Start the console with the FreeMcBoot memory card, open wLaunchELF
   (uLaunchELF), go to `FileBrowser` → `mass:/` → `NGCD/` and run
   `ngcd-ps2.elf`.
4. For an item in the FreeMcBoot menu: `FMCB Configurator` → `Configure
   OSDSYS options` → `Configure Item`, path `mass:/NGCD/ngcd-ps2.elf`, then
   `Save CNF to MC0`.

HDD, MX4SIO and `host:` (PCSX2 or ps2link) work the same way: the files are
read from the ELF's folder.

A red screen means that a required file (`NEOCD.BIN`, `TOC.BIN`,
`DATA.ISO`) is missing or that the BIOS has the wrong size. No picture or
no sync on the TV: put `video=480i` in `NGCD.CFG` (240p is the default and
has not been tried yet); add `tv=pal` if a PAL TV does not take 60 Hz.

### Logs

The debug build writes to the EE console:

    ngcd: 300 frames, 297 shown, cdda underruns 0, ym underruns 0, sound drops 0

On a console the log can be seen with ps2link: start ps2link from FreeMcBoot
or wLaunchELF, then on the PC, in the game's folder:

    ps2client -h <console IP> execee host:ngcd-ps2.elf

The game's files are then read from the PC over the network. In PCSX2 the
log is in the EE console window.

## NGCD.CFG

A text file in the game's folder, `key=value` lines, for example:

    cpu=18
    region=us
    video=480i

| key | values | default |
|---|---|---|
| `region` | `jp`, `us`, `eu` | `jp` |
| `video` | `240p`, `480i` | `240p` |
| `tv` | `ntsc`, `pal` | `ntsc` |
| `filter` | `0`, `1` | `0` |
| `stick` | `0`, `1` | `1` |
| `cpu` | `12`, `15`, `18`, `21`, `24` | `12` |
| `idle` | `0`, `1` | `1` |

`region` — the console's region, read by the BIOS: the BIOS and the games
show that region's title, language and content.

`video` — `240p`: the 320x224 picture with one TV line per Neo Geo line, as
on the real console (each pixel two dots wide). `480i`: 640x448 interlaced,
every pixel 2x2.

`tv` — `ntsc` (default): 60 Hz, the games' own speed. `pal`: 50 Hz; the
games then run at 50 frames a second, 5/6 of their speed.

`filter` — `filter=1` smooths the picture (bilinear); by default the
pixels are sharp.

`stick` — `stick=0` leaves the left analog stick unused (by default it
moves like the D-pad, past half of its travel).

`cpu` — 68000 clock in MHz: 12 (the real console), 15, 18, 21 or 24.
Games that slow down in heavy scenes on the real hardware (Metal Slug) slow
down less; timers, video, sound and CD keep the real speed. It costs EE
time, so it is only worth trying once 12 MHz runs at full speed.

`idle` — `idle=0` turns off skipping of the 68000's wait loops (on by
default): while a game waits for the next frame in a short loop that only
reads RAM, the emulator does not run it until an interrupt arrives.

## Controls

| DualShock | Neo Geo |
|---|---|
| D-pad or left stick | joystick |
| Cross / Circle / Square / Triangle | A / B / C / D |
| Start | Start |
| Select | Select |
| L1+R1+L2+R2+Start+Select | quit (back to the launcher) |

The second pad is player 2. The CDZ's 8 KB backup RAM (saves) goes to the
memory card in slot 1, `mc0:/NGCDPS2/BACKUP.BIN`, about 2 s after it
changes; it is shared by all games, as on the CDZ.

## Source layout

```
src/core/      Neo Geo CD core, portable C (shared with the Dreamcast build)
  ngcd.c         frame loop, timers, interrupts, sound
  mem.c          68000 memory map, DMA, 0xE00000 window
  cd.c           LC8951, drive protocol, TOC
  video.c        software renderer (and the Dreamcast build's rectangle builder)
  ym2610.c       YM2610 (MAME fm.c via neocd_libretro, converted to C)
  nga.h          CD audio format
src/ps2/       PlayStation 2 front end
  main_ps2.c     main loop, files, CD audio, sound, video, pads, memory card, NGCD.CFG
  cpu_famec.c    68000 on FAME C
src/cpu/famec/ FAME C (PicoDrive's copy)
host/          PC build (Musashi) and the C Z80 core used on the PS2 too
tools/         ngcdconv
```

## PC build

`host/` builds the same core for a PC with Musashi and the same C Z80. It
writes frames (PPM) and audio (WAV); this is how the core is compared with
a reference.

    cd host && make
    ./ngcd_host neocd.bin ng-lo.rom|- disc_folder frames every out_dir [script]

`script` is a list of button presses `frame:keys,frame:keys`, keys
`U D L R a b c d S s` (S Start, s Select), each held for 8 frames.
`NGCD_CPU_MHZ=18` in the environment sets the 68000 clock, `NGCD_IDLE=0`
turns wait-loop skipping off.

FAME C was compared with Musashi in the PC build: the pictures are the same
until the games drift apart in timing (the two cores count some
instructions' cycles differently).

## Known limitations

- The picture is drawn by the EE (software renderer). Drawing sprites with
  the GS instead, as the Dreamcast build does with its PVR, would take
  1–3 M instructions a frame off the EE; not done yet.
- Raster effects: sprite tables rewritten during the picture (Riding Hero's
  road, Neo Turf Masters' lake) are followed; tile numbers, palettes and
  the fix layer changed in the middle of the picture show as they are at
  the end of the frame.
- CD audio and the data track are read in the main loop (small reads of
  the `.NGA` file each frame, 64 KB of the data track at a time).
- Games cannot be read from an original Neo Geo CD disc in the PS2's drive
  yet; the files have to be converted with `ngcdconv`.

## Credits and licences

See [THIRD_PARTY.md](THIRD_PARTY.md) for the full list and terms.

- Machine behaviour after NeoCD / neocd_libretro by Fabrice Martinez
  (LGPL-3.0); the core is rewritten in C.
- YM2610 — Jarek Burczynski and Tatsuyuki Satoh (MAME `fm.c`) via
  neocd_libretro, GPL-2.0-or-later.
- FAME C — Oscar Orallo Pelaez, with PicoDrive's changes (notaz, irixxxx).
- Z80 — Nicolas Allemand (superzazu), MIT.
- Musashi (PC build only) — Karl Stenerud, MIT.
- NeoCD-PS2 by Olivier "Evilo" Biot showed that the Neo Geo CD runs on the
  EE; its code is not used here.

The project is distributed under the GNU General Public License v3.0 or
later ([LICENSE](LICENSE)). FAME keeps its own terms (see THIRD_PARTY.md).
