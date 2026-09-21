# PICCLAUS — ILE COBOL PICTURE Clause Reference

An interactive IBM i program that displays a scrollable reference of all COBOL
`PIC` clause forms, with a brief description of each.  Run it from a 5250
session whenever you need a quick reminder of picture string syntax.

## What it does

- Presents a full-screen 5250 display (24×80) listing PIC clause examples
  paired with plain-English descriptions
- 60 entries covering every major category: alphabetic, alphanumeric, unsigned
  and signed numeric, implied decimal, scaling, floating-point, binary,
  edited/formatted output, and level 66/88 special forms
- 18 entries visible per page; **PageDown / PageUp** scroll through the list
- **F3** exits the program
- Page `n of N` counter and a status message line for scroll-boundary hints

## Source files

| File                                       | Description                                                |
| ------------------------------------------ | ---------------------------------------------------------- |
| [`src/picclaus.cblle`](src/picclaus.cblle) | ILE COBOL source — program logic and PIC clause data table |
| [`src/picclaus.dspf`](src/picclaus.dspf)   | DDS display file — 24×80 screen layout and key definitions |
| [`Makefile`](Makefile)                     | Build script for IBM i (runs via PASE `make`)              |

## PIC clause categories covered

| Category                                   | Examples                                                   |
| ------------------------------------------ | ---------------------------------------------------------- |
| Alphabetic (`A`)                           | `PIC A(10)`, `PIC A(2)`                                    |
| Alphanumeric (`X`)                         | `PIC X(30)`, `PIC X(1)`, `PIC X(80)`                       |
| Unsigned numeric (`9`)                     | `PIC 9(5)`, `PIC 9(7)V99`                                  |
| Signed numeric (`S9`)                      | `PIC S9(5)`, `PIC S9(9)V99 COMP-3`, `PIC S9(9) COMP-4`     |
| Implied decimal (`V`)                      | `PIC 9(5)V9(2)`, `PIC S9(7)V9(4)`                          |
| Scaling (`P`)                              | `PIC 9(3)PPP`, `PIC PPP9(3)`                               |
| Floating-point (`COMP-1`/`COMP-2`)         | `PIC S9(6)V9 COMP-1`, `PIC S9(16)V9(7) COMP-2`             |
| Native binary (`COMP-5`)                   | `PIC S9(4) COMP-5`, `PIC S9(9) COMP-5`                     |
| Edited — zero suppression (`Z`)            | `PIC ZZZZ9`, `PIC Z(6)9.99`                                |
| Edited — asterisk fill (`*`)               | `PIC ***9.99`, `PIC *(6)9.99`                              |
| Edited — fixed dollar (`$`)                | `PIC $ZZZ,ZZ9.99`, `PIC $9(7).99`                          |
| Edited — floating dollar (`$$`)            | `PIC $$(6)9.99`                                            |
| Edited — sign symbols (`+`/`-`/`CR`/`DB`)  | `PIC +ZZZ9.99`, `PIC ZZZ9.99CR`, `PIC ZZZ9.99DB`           |
| Edited — comma, period, slash, blank, zero | `PIC ZZZ,ZZ9.99`, `PIC 99/99/9999`, `PIC 9(4)B9(2)B9(2)`   |
| Level 66 RENAMES                           | `66 NEWNAME RENAMES OLDNAME`                               |
| Level 88 condition names                   | `88 FLAG-ON VALUE 'Y'`                                     |

## Prerequisites

- IBM i with ILE COBOL (`CRTBNDCBL`) available
- PASE environment with `make` (from the IBM i Open Source package repository)
- A target library that already exists on the system

## Building

The [`Makefile`](Makefile) compiles both objects from the IFS stream files.
Run it from the `picclaus/` directory inside a PASE shell:

```sh
# Build everything into MYLIB
make all LIB=MYLIB

# Build display file only
make dspf LIB=MYLIB

# Build COBOL program only (also builds the display file if needed)
make cobol LIB=MYLIB

# Remove compiled objects from the library
make clean LIB=MYLIB
```

### Optional parameters

| Parameter | Default      | Description                                       |
| --------- | ------------ | ------------------------------------------------- |
| `LIB`     | *(required)* | Target library for compiled objects               |
| `SRCPF`   | `QDDSSRC`    | Source physical file used to stage the DDS member |
| `SRCDIR`  | `src`        | Directory containing the source files             |

## Running

After a successful build, call the program from a 5250 command line:

```cl
CALL LIB/PICCLAUS
```

### Key bindings

| Key          | Action                               |
| ------------ | ------------------------------------ |
| **PageDown** | Scroll forward one page (18 entries) |
| **PageUp**   | Scroll back one page                 |
| **F3**       | Exit the program                     |
