# cobolinfo Plan

## Top-Level Overview

Re-engineer the existing `picclaus/` module into a new `cobolinfo/` module at the repo root.
The goal is to separate **display logic** from **content** so that future COBOL reference topics
can be added as additional exported procedures in a service program without touching the display program.

### Scope
- New folder: `cobolinfo/` (alongside `picclaus/`, same flat repo layout)
- New display program: `COBOLINFO` (`cobolinfo.cblle`) — handles all 5250 screen I/O
- New service program: `COBINFOSRV` (`cobinfosrv.cblle`) — supplies content via exported procedures
- First exported procedure: `PICCLAUS-DATA` — returns the 60-entry PIC clause table
- New display file: `cobolinfo.dspf` — header row changed to dynamic `HDRTOPIC` output field
- New binding directory definition: `cobinfobd.bnd` — registers `COBINFOSRV` entries
- New `Makefile` covering all build steps
- New `README.md`

### Non-goals
- No new PIC clause entries (same 60 as in `picclaus`)
- No change to page size, scroll logic, or indicator assignments
- No GUI / web layer

---

## Sub-Task 1 — Create the folder skeleton and README

### Intent
Establish the `cobolinfo/` directory with a `src/` sub-folder and a `README.md`
so all subsequent sub-tasks have a home.

### Expected Outcomes
- `cobolinfo/` exists at repo root
- `cobolinfo/src/` exists
- `cobolinfo/README.md` exists with a short description

### Todo List
1. Create `cobolinfo/src/` (an empty placeholder file is sufficient)
2. Write `cobolinfo/README.md` describing the program, its two-component design,
   build prerequisites, and how to call it

### Relevant Context
- Follow the `picclaus/README.md` style (95 lines, table of PIC categories,
  prerequisites, build examples)

### Status
[x] done

---

## Sub-Task 2 — Create the DDS display file `cobolinfo.dspf`

### Intent
Produce a display file nearly identical to `picclaus/src/picclaus.dspf` but with
one structural change: replace the static "COBOL PICTURE Clause" header text with
a 30-character output field named `HDRTOPIC` so the main program can write the
topic title at runtime.

### Expected Outcomes
- `cobolinfo/src/cobolinfo.dspf` exists
- Record format is named `COBSCR` (matches the new program name)
- Row 1 col 2 has `HDRTOPIC` (30A, output, `COLOR(WHT)`, `DSPATR(HI)`) instead of static text
- All other fields (`PAGENBR`, `TOTPAGES`, `MSGFLD`, `LINE01`–`LINE18`) are identical
  to the originals in position, size, and colour
- Function key bindings unchanged: CA03=03, PAGEDOWN=25, PAGEUP=26

### Todo List
1. Copy structure from `picclaus/src/picclaus.dspf`
2. Rename record format from `PICSCR` to `COBSCR`
3. Replace static row-1 "COBOL PICTURE Clause" text with `HDRTOPIC` 30A output field
4. Adjust the static "Page:" label in row 2 if its column position shifted
5. Leave all other field definitions and attributes unchanged

### Relevant Context
- `picclaus/src/picclaus.dspf` lines 1–60
- `HDRTOPIC` will be populated by `cobolinfo.cblle` before each WRITE cycle
- Record format name `COBSCR` must match the `FORMAT IS 'COBSCR'` clause in `cobolinfo.cblle`

### Status
[x] done

---

## Sub-Task 3 — Create the service program source `cobinfosrv.cblle`

### Intent
Write an ILE COBOL module that exports a single procedure, `PICCLAUS-DATA`, which
accepts a BY REFERENCE group-item and fills it with the count and 60-entry PIC clause
table.  All 60 MOVE statements are lifted verbatim from `picclaus.cblle` `INIT-TABLE`.

### Expected Outcomes
- `cobolinfo/src/cobinfosrv.cblle` exists
- `PROGRAM-ID. COBINFOSRV`
- DATA DIVISION contains the group-item definition that matches the caller's parameter:
  - Count field: `PIC 9(3) COMP-4`
  - Table: 60 occurrences of 77-char items (27-char example + 50-char description)
- PROCEDURE DIVISION contains one paragraph (`PICCLAUS-DATA`) that:
  1. Sets count to 60
  2. Executes the 60 MOVE pairs (lifted from `INIT-TABLE` in `picclaus.cblle`)
  3. GOBACK
- The procedure is declared with `PROCEDURE DIVISION GIVING` or via a prototype
  that matches the BY REFERENCE parameter convention used in `cobolinfo.cblle`

### Todo List
1. Write IDENTIFICATION and ENVIRONMENT DIVISION (IBM-I; no workstation file)
2. Define the shared group-item layout in WORKING-STORAGE (or LINKAGE SECTION)
3. Write `PICCLAUS-DATA` PROCEDURE paragraph; copy MOVE statements from
   `picclaus/src/picclaus.cblle` lines 236–527 (`INIT-TABLE`)
4. Add `GOBACK` at end of procedure

### Relevant Context
- Source of MOVE statements: `picclaus/src/picclaus.cblle` lines 236–527
- Group-item layout must be **identical** in both the service program (LINKAGE SECTION)
  and the calling program (WORKING-STORAGE), so define it once in terms agreed with Sub-Task 4
- ILE COBOL service program modules use `PROCESS NOMAIN` (or equivalent) and must not
  have a main procedure entry

### Status
[x] done

---

## Sub-Task 4 — Create the main display program `cobolinfo.cblle`

### Intent
Write the new interactive program `COBOLINFO` that:
1. Calls `PICCLAUS-DATA` from `COBINFOSRV` to retrieve the PIC clause table
2. Drives the 5250 scroll interface using `cobolinfo.dspf`
3. Writes a topic title into `HDRTOPIC` before each screen write

All scroll, paging, and screen logic is preserved from `picclaus.cblle`; only the
data-loading paragraph is replaced by the service program call.

### Expected Outcomes
- `cobolinfo/src/cobolinfo.cblle` exists
- `PROGRAM-ID. COBOLINFO`
- FILE-CONTROL maps to `COBSCRF` (workstation file using `cobolinfo.dspf` object)
- WORKING-STORAGE includes:
  - The same group-item layout as the service program linkage section
    (count + 60-entry table)
  - `WS-HDRTOPIC PIC X(30)` initialised to `'COBOL PICTURE Clauses'`
  - All existing scroll/page fields from `picclaus.cblle`
- PROCEDURE DIVISION calls `PICCLAUS-DATA` BY REFERENCE instead of executing `INIT-TABLE`
- `FILL-SCREEN` moves `WS-HDRTOPIC` to the `HDRTOPIC` screen field before WRITE
- All other paragraphs (`SCROLL-LOOP`, `PAGE-DOWN`, `PAGE-UP`, `CALC-PAGES`,
  `WRITE-SCREEN`, `READ-SCREEN`) are functionally identical to `picclaus.cblle`
- Bound with `COBINFOBD` binding directory (specified in `CRTPGM` or in `CRTBNDCBL` BNDDIR)

### Todo List
1. Write IDENTIFICATION + ENVIRONMENT DIVISION; rename workstation file reference to `COBSCRF`
2. Copy DATA DIVISION from `picclaus.cblle`; rename `PICSCRF`→`COBSCRF`; add `WS-HDRTOPIC`
3. Add group-item layout matching `cobinfosrv.cblle` LINKAGE SECTION
4. Replace `INIT-TABLE` call in MAIN-PARA with `CALL PROCEDURE 'PICCLAUS-DATA'
   USING BY REFERENCE <group-item>`
5. In `WRITE-SCREEN` (or `FILL-SCREEN`), add MOVE `WS-HDRTOPIC` TO `COBSCR-HDRTOPIC`
6. Rename all `PICSCR`-prefixed field references to `COBSCR`-prefixed names
7. Rename FORMAT literals from `'PICSCR'` to `'COBSCR'`
8. Verify `BNDDIR` clause or compile option points to `COBINFOBD`

### Relevant Context
- `picclaus/src/picclaus.cblle` is the primary reference; preserve paragraphs verbatim
  except where noted above
- The CALL PROCEDURE syntax for ILE COBOL service programs:
  `CALL PROCEDURE 'PROCNAME' USING BY REFERENCE data-item`
- `WS-HDRTOPIC` value `'COBOL PICTURE Clauses'` is the first topic title; future topics
  will be additional values written before calling a different service procedure

### Status
[x] done

---

## Sub-Task 5 — Create the binder language source `cobinfobd.bnd`

### Intent
Define the binding directory entry that registers `COBINFOSRV` as a module source
so `COBOLINFO` can resolve `PICCLAUS-DATA` at bind time.

### Expected Outcomes
- `cobolinfo/src/cobinfobd.bnd` exists
- Contains binder language that adds `COBINFOSRV` (type `*SRVPGM`) to `COBINFOBD`

### Todo List
1. Write binder language file with `STRPGMEXP` / `EXPORT SYMBOL('PICCLAUS-DATA')` / `ENDPGMEXP`

### Relevant Context
- IBM i binder language reference for `STRPGMEXP`, `EXPORT`, `ENDPGMEXP`
- The binding directory object `COBINFOBD` is created by the Makefile (Sub-Task 6);
  this file only defines what is exported from `COBINFOSRV`

### Status
[x] done

---

## Sub-Task 6 — Create the Makefile

### Intent
Write a `Makefile` that builds all `cobolinfo` artifacts in dependency order:
DDS display file → service program module → bind service program → main program.

### Expected Outcomes
- `cobolinfo/Makefile` exists
- Targets: `all`, `dspf`, `srvpgm`, `cobol`, `clean`, `help`
- `all` depends on `dspf` → `srvpgm` → `cobol` in that order
- `srvpgm` target:
  1. Compiles `cobinfosrv.cblle` with `CRTSQLCBLI` or `CRTCBLMOD` → produces module `COBINFOSRV`
  2. Creates `COBINFOBD` binding directory with `CRTBNDDIR`
  3. Adds `COBINFOSRV *SRVPGM` entry with `ADDBNDIRE`
  4. Runs `CRTSRVPGM` using `cobinfobd.bnd` binder language to produce `COBINFOSRV *SRVPGM`
- `cobol` target compiles `cobolinfo.cblle` with `CRTBNDCBL` specifying `BNDDIR(LIB/COBINFOBD)`
- `clean` deletes `*PGM COBOLINFO`, `*SRVPGM COBINFOSRV`, `*BNDDIR COBINFOBD`, `*DSPF COBSCRF`
- Required parameter: `LIB`; optional: `SRCPF`, `SRCDIR` (same pattern as `picclaus/Makefile`)

### Todo List
1. Copy structure from `picclaus/Makefile`; rename variables and targets
2. Add `srvpgm` target with `CRTCBLMOD`, `CRTBNDDIR`, `ADDBNDIRE`, `CRTSRVPGM` steps
3. Update `cobol` target to pass `BNDDIR` to `CRTBNDCBL`
4. Update `clean` to include service program and binding directory objects
5. Update `help` text

### Relevant Context
- `picclaus/Makefile` lines 1–165 — base structure to follow
- IBM i commands: `CRTCBLMOD`, `CRTSRVPGM SRCSTMF(...)`, `CRTBNDDIR`, `ADDBNDIRE`,
  `CRTBNDCBL BNDDIR(LIB/COBINFOBD)`
- The binder language file `cobinfobd.bnd` is passed to `CRTSRVPGM` via `SRCSTMF`

### Status
[x] done

---

## Implementation Notes

- **Sub-tasks 1–6 must be done in order** — each file depends on decisions made in the prior one.
- **Group-item layout** must be agreed between Sub-Tasks 3 and 4 before either is coded;
  the data layout is the interface contract between the two modules.
- **DDS record format name `COBSCR`** must be consistent across Sub-Tasks 2, 4, and 6.
- **No changes to `picclaus/`** — it remains untouched as the original example.
