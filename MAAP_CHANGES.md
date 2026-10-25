# Changes from upstream

Upstream: https://github.com/geoschem/hemco, tag `3.13.0` at commit 870e8802284d9e180b1a2b017ae6f5616cbbaedd (2026-09-15), by the Developers of GEOS-Chem, HEMCO, and Related Software (see `AUTHORS.txt`), MIT License (`LICENSE.txt`). This is the HEMCO version that GEOS-Chem Classic 14.8.0 pins as its `src/HEMCO` submodule.

This copy starts from upstream's file tree at 870e880 without its git history. It is unchanged except for the item below. Left out: `.github/` (issue templates and CI workflows). The `docs/source/geos-chem-shared-docs` submodule still points to geoschem/geos-chem-shared-docs at f882128, as upstream.

It is used as the `src/HEMCO` submodule of https://github.com/gchang/geos-chem-classic, whose `MAAP_CHANGES.md` describes the whole bundle.

## 2026-10-25: extensions allow data paths longer than 255 characters

The VOLCANO, PARANOX and LightNOx extensions (`src/Extensions/hcox_volcano_mod.F90`, `hcox_paranox_mod.F90`, `hcox_lightnox_mod.F90`) kept the input file path and the file status message in 255-character strings. The message is the full path plus a prefix such as `HEMCO (PARANOX): REQUIRED FILE NOT FOUND`, written with an internal `WRITE( MSG, 300 )`, so once the data root passed about 140 characters the write overflowed and the model stopped with `Fortran runtime error: End of record`. With GEOS-Chem Classic's fullchem simulation this happened in the dry run: in VOLCANO with a 144-character data root, and in PARANOX with a 190-character root. A normal run writes the same `Opening ...` line and fails the same way. Paths longer than 255 characters were also cut off silently.

These strings are now 1023 characters, the length HEMCO's file reader already uses for source file paths and messages (`srcFile`, `MSG` in `src/Core/hcoio_read_std_mod.F90`) and the length of configuration option values (`OPTLEN`):

- VOLCANO: `FileName` and `ClimFile` (instance type), and `ThisFile`, `MSG`, `FileMsg` in `ReadVolcTable`. `ThisLine` (one line of the table) stays at 255.
- PARANOX: `LutDir` (instance type), `FILENAME` in `READ_PARANOX_LUT_NC` and `READ_PARANOX_LUT_TXT`, and `MSG`, `FileMsg` in `READ_LUT_NCFILE` and `READ_LUT_TXTFILE`.
- LightNOx: `MSG`, `FILENAME`, `FileMsg` in `HCOX_LightNOx_Init`.

Nothing else changes, so emissions are identical, and runs with short paths behave exactly as before. HEMCO's core file reader already handled long paths; the only other places that print a file status are in `src/Core` and write straight to the log unit, not into a fixed-length string.

## Check

Done on 2026-10-25 as part of the GEOS-Chem Classic bundle (details in its `MAAP_CHANGES.md`): with a 190-character data root, the fullchem dry run of upstream 14.8.0 stops with `End of record` in PARANOX, and with this copy it runs to the end and lists volcano table paths of 262 to 267 characters in full. TransportTracers runs (1 hour and 24 hours) with this copy and the 190-character root give output identical to upstream 14.8.0 with a short root.
