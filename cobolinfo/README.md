# COBOLINFO - ILE COBOL Reference Program

`COBOLINFO` is an interactive 5250 reference program for IBM i that displays
scrollable COBOL reference information organised by topic.  It is a
re-engineering of the original `PICCLAUS` example, separating the **display
logic** (the `COBOLINFO` main program) from the **content** (the `COBINFOSRV`
service program) so that new topics can be added without touching the display
code.

---

## Architecture

```
COBOLINFO (*PGM)          -- interactive 5250 display program
    |
    +-- calls COBINFOSRV      (exported ILE procedure in COBINFOSRV *SRVPGM)
    |       fills WS-TOPIC group-item: count + 60-entry table
    |
    +-- drives COBINFODSP (*FILE, compiled from cobolinfo.dspf)
            COBSCR record format
            HDRTOPIC output field (dynamic topic header)
```

In ILE COBOL, only the `PROGRAM-ID` of the outermost program in a compilation
unit is exported as an ILE procedure symbol.  Each topic in `COBINFOSRV` is
therefore a separate `PROGRAM-ID` (compilation unit) within the same source
file, and each is listed in `cobinfobd.bnd`.

Binding is resolved through the `COBINFOBD` binding directory which lists
`COBINFOSRV *SRVPGM` as an entry.

---

## Topics Available

| PROGRAM-ID / Symbol | Topic                       | Entries |
|---------------------|-----------------------------|---------|
| COBINFOSRV          | COBOL PICTURE Clauses       | 60      |

*(More topics will be added as additional `PROGRAM-ID` compilation units in*
*`cobinfosrv.cblle`, each listed in `cobinfobd.bnd`.)*

---

## PICTURE Clause Categories

| # | Category                      | Entries |
|---|-------------------------------|---------|
| 1 | Alphabetic (A)                | 3       |
| 2 | Alphanumeric (X)              | 4       |
| 3 | Unsigned Numeric (9)          | 5       |
| 4 | Signed Numeric (S9)           | 6       |
| 5 | Implied Decimal (V)           | 3       |
| 6 | Scaling Factor (P)            | 3       |
| 7 | Float (COMP-1 / COMP-2)       | 3       |
| 8 | Native Binary (COMP-5)        | 3       |
| 9 | Edited: Zero Suppress (Z)     | 3       |
|10 | Edited: Asterisk Fill (*)     | 3       |
|11 | Edited: Fixed Dollar ($)      | 3       |
|12 | Edited: Floating Dollar ($$)  | 2       |
|13 | Edited: Sign Symbols          | 6       |
|14 | Edited: Comma / Period        | 3       |
|15 | Edited: Slash / Blank         | 3       |
|16 | Edited: Zero Insertion (0)    | 2       |
|17 | Level 66 RENAMES              | 2       |
|18 | Level 88 Conditions           | 3       |

Total: 60 entries displayed 18 per page (4 pages).

---

## Source Files

| File                    | Object created                  | Purpose                          |
|-------------------------|---------------------------------|----------------------------------|
| `src/cobolinfo.dspf`    | `COBINFODSP *FILE`              | DDS display file                 |
| `src/cobinfosrv.cblle`  | `COBINFOSRV *MODULE` + `*SRVPGM`| Service program - content only   |
| `src/cobinfobd.bnd`     | staged into `QBNDSRC(COBINFOBD)`| Binder language - export symbols |
| `src/cobolinfo.cblle`   | `COBOLINFO *PGM`                | Interactive display program      |

The `srvpgm` build target also creates the `COBINFOBD *BNDDIR` binding directory and
adds `COBINFOSRV *SRVPGM` to it.

---

## Prerequisites

- IBM i with ILE COBOL licensed (5770-CB1)
- PASE environment with `make` (`yum install make`)
- Target library must exist before building
- A source physical file for binder language source must exist in the target
  library (default name `QBNDSRC`); the Makefile creates it automatically if
  absent via `CRTSRCPF`

---

## Building

```sh
# Build all objects (display file, service program, main program)
make all LIB=MYLIB

# Use a non-default source PF for binder language source
make all LIB=MYLIB BNDSRCPF=MYBNDSRC

# Build display file only
make dspf LIB=MYLIB

# Build service program and binding directory only
make srvpgm LIB=MYLIB

# Build main COBOL program only (requires dspf + srvpgm already built)
make cobol LIB=MYLIB

# Delete all compiled objects
make clean LIB=MYLIB

# Show build help
make help
```

---

## Running

```cl
CALL LIB/COBOLINFO
```

Use `F3` to exit, `PgDn` / `PgUp` to scroll through the reference.
