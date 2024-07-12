# Xcode Project for debugging MAGIC

[MAGIC](http://opencircuitdesign.com/magic/) is a VLSI EDA tool suite, available on Github at https://github.com/RTimothyEdwards/magic/.

In order to figure out its `configure`/`Make` based build process, I recreated the build process in Xcode.

## Learnings

### MAGIC build process overview

- Magic has multiple modules (e.g. `commands`, `magic`, `extract`, ...)
- For each module (Example `extract`)
   - it compiles each `*.c` file into a corresponding `*.o` file (Example: `gcc -o ExtBasic.o -c ExtBasic.c`)
   - it merges all `*.o` files into a corresponding `lib<module>.o` file (Example: `ld -r ExtArray.o ExtBasic.o ... -o libextract.o`)
- Shared library `tclmagic.dylib`
   - All `lib<module>.o` files are linked into a shared library `tclmagic.dylib` (`gcc -o tclmagic.dylib -dynamiclib ../extract/libextract.o ../commands/libcommands.o ...`)
   - The shared library links against `-ltkstub8.6` and `-ltclstub8.6` and X11 related libraries
- There are 2 binaries `magicexec` and `magicdnull`
   - They only link aginst `-ltk8.6` and `-ltcl8.6`, and the shared library gets loaded from the main TCL script `magic.tcl` using `load -lazy ...` (which is using `dlopen`)
 
### Hard coded strings via compile flags

- `magicexec` will search for a TCL script `magic.tcl`, whose location `TCL_DIR` is hard coded via compile flags `-DTCL_DIR=\"/usr/local/lib/magic/tcl\"`
- amongst other (less important) examples are of such hard coded compiler flag strings are `CAD_DIR`, and for the version information (`MAGIC_REVISION`, `MAGIC_VERSION`, `MAGIC_COMMIT`)

## Dependencies

- XQuartz installed via HomeBrew: `brew install xquartz`
- It is expected that TCL/TK 8.6 has been compiled to `/usr/local/opt/tcl`
  - Tcl: https://prdownloads.sourceforge.net/tcl/tcl8.6.10-src.tar.gz
  - Tk: https://prdownloads.sourceforge.net/tcl/tk8.6.10-src.tar.gz
