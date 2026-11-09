# Third-party code and terms

| Component | Files | Author | Terms |
|---|---|---|---|
| NeoCD / neocd_libretro (behaviour reference) | `src/core/*.c` follow its design; rewritten in C | Fabrice Martinez | LGPL-3.0 |
| YM2610 (MAME `fm.c`) | `src/core/ym2610.c`, `ym2610.h` | Jarek Burczynski, Tatsuyuki Satoh (via neocd_libretro) | GPL-2.0-or-later, see the file header |
| FAME C 1.24 (68000) | `src/cpu/famec/` (PicoDrive's copy; Neo Geo vectors via `famec_irq_vector`, PicoDrive's Sega CD hack off) | Oscar Orallo Pelaez; PicoDrive changes by notaz, irixxxx | the FAME author's terms, `src/cpu/famec/fame.html` |
| Z80 | `host/z80.c`, `host/z80.h` | Nicolas Allemand (superzazu) | MIT, `host/LICENSE.z80` |
| Musashi (68000, PC build only) | `host/musashi/` | Karl Stenerud | MIT, `host/musashi/LICENSE.txt` |

The rest of the code (`src/ps2/`, `src/core/` glue, `tools/`, `host/main.c`,
`host/cpu_musashi.c`, `host/z80_zazu.c`) is part of ngcd and distributed
under the GNU GPL v3.0 or later (`LICENSE`).

Note: fame.html allows FAME to be distributed freely in unmodified form; the
C version here carries PicoDrive's changes and two small ones of ours (the
vector of the pending interrupt, the Sega CD TAS hack switched off), as
PicoDrive itself ships a modified FAME C. Anyone redistributing this
commercially should check the FAME terms.

No BIOS, ROM or game data is included. `neocd.bin` (the CDZ BIOS) and the
games must come from the user's own hardware and discs.
