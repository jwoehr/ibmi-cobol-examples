# cobolinfo Verb Topic Plan

## Top-Level Overview

Extend the existing `cobolinfo/` module with two coordinated changes:

1. **Service program expansion** — add a new exported ILE procedure `COBVERBS`
   (a second `PROGRAM-ID` compilation unit in `cobinfosrv.cblle`) that fills the
   shared topic array with COBOL verb entries, formatted as 27-char verb/syntax
   column + 50-char description column, using consecutive pairs of lines: the
   first line is the verb name plus a short description, and the second is an
   indented usage example with a brief usage note.

2. **Topic switching in the main program** — add `F5=Next Topic` to `cobolinfo.dspf`
   and `cobolinfo.cblle` so the user can cycle forward through the topic list
   (PIC Clauses → Verbs → PIC Clauses → …).  No display-logic restructuring is
   needed beyond the new key and the topic-cycling state.

3. **Array size expansion** — the shared `WS-TOPIC` interface grows from
   `OCCURS 60 TIMES` to `OCCURS 120 TIMES` to accommodate the verb topic, which
   requires parallel edits to both the service program LINKAGE SECTION and the
   main program WORKING-STORAGE.

4. **Binder update** — `cobinfobd.bnd` gains an `EXPORT SYMBOL('COBVERBS')` line.

5. **README update** — `cobolinfo/README.md` documents the new topic, the new
   key binding, and the expanded array size.

### Scope
- `cobolinfo/src/cobinfosrv.cblle` — expand OCCURS; add `COBVERBS` compilation unit
- `cobolinfo/src/cobolinfo.dspf` — add CA05 key and `F5=Next Topic` legend
- `cobolinfo/src/cobolinfo.cblle` — add topic-cycle state, F5 handler; expand OCCURS
- `cobolinfo/src/cobinfobd.bnd` — add COBVERBS export symbol
- `cobolinfo/README.md` — update topics table, controls, array note

### Non-goals
- No changes to scroll logic, page size, or indicator assignments beyond F5
- No GUI / web layer
- No new PIC clause entries

---

## Sub-Task 1+2 — Edit cobinfosrv.cblle: expand array and add COBVERBS

### Intent
Perform both service-program edits in a single pass of `cobinfosrv.cblle`:
(a) Grow `OCCURS 60 TIMES` → `OCCURS 120 TIMES` in the existing COBINFOSRV
    LINKAGE SECTION so the interface contract accommodates the new verb topic.
(b) Append the new `COBVERBS` compilation unit after `END PROGRAM COBINFOSRV.`
    — a complete second PROGRAM-ID that fills the shared array with COBOL verb
    entries using the same WS-LOAD pattern.

Doing both changes in one pass avoids a partial-compile state and keeps the
diff reviewable as a single logical change to this file.

### Expected Outcomes
- `cobinfosrv.cblle` LINKAGE SECTION `WS-ENTRY OCCURS 60 TIMES` → `OCCURS 120 TIMES`
- Header comment updated to reflect 120-entry maximum
- Existing `COBINFOSRV` PROGRAM-ID procedure and all its `MOVE … TO WS-ENTRY(n)`
  statements are untouched
- A new `IDENTIFICATION DIVISION … PROGRAM-ID. COBVERBS` compilation unit exists
  at the end of `cobinfosrv.cblle`
- COBVERBS declares the same LINKAGE SECTION layout (`WS-TOPIC-COUNT` +
  `WS-ENTRY OCCURS 120 TIMES`)
- COBVERBS uses the same WS-LOAD helper pattern (27-char WS-EX + 50-char WS-DC)
  in its own WORKING-STORAGE
- COBVERBS PROCEDURE DIVISION sets `WS-TOPIC-COUNT` and populates `WS-ENTRY(1..N)`
- File ends with `END PROGRAM COBVERBS.`

#### Verb entry format (two-line pairs per verb)
Each category begins with a separator header entry (dashes + label), then for
each verb two consecutive entries:
- **Line 1** — WS-EX = verb name (e.g. `MOVE`), WS-DC = short description of what the verb does
- **Line 2** — WS-EX = indented syntax/usage example (e.g. `  MOVE SRC TO DEST`), WS-DC = brief usage note

#### Verb categories and entries to include

| Category          | Verbs                                            |
|-------------------|--------------------------------------------------|
| Data Movement     | MOVE, INITIALIZE, SET, STRING, UNSTRING          |
| Arithmetic        | ADD, SUBTRACT, MULTIPLY, DIVIDE, COMPUTE         |
| Control Flow      | PERFORM, IF/ELSE/END-IF, EVALUATE, GO TO, STOP   |
| I/O               | OPEN, CLOSE, READ, WRITE, REWRITE, DELETE        |
| Table Handling    | SEARCH, SEARCH ALL, SORT, MERGE                  |
| Program Linkage   | CALL, CANCEL, GOBACK, EXIT PROGRAM               |
| Exception / Debug | INSPECT, ON EXCEPTION, ON SIZE ERROR, DISPLAY    |

### Todo List
1. In `cobinfosrv.cblle` LINKAGE SECTION (line 56), change `OCCURS 60 TIMES`
   → `OCCURS 120 TIMES`; update the header comment (lines 17–18) to say 120
2. After `END PROGRAM COBINFOSRV.` (line 296), append the new compilation unit:
   a. IDENTIFICATION + ENVIRONMENT DIVISION (mirror COBINFOSRV header style)
   b. WORKING-STORAGE with WS-LOAD helper (identical 27+50 layout)
   c. LINKAGE SECTION with WS-TOPIC (`WS-TOPIC-COUNT` + `WS-ENTRY OCCURS 120 TIMES`)
   d. PROCEDURE DIVISION USING BY REFERENCE WS-TOPIC
   e. Set WS-TOPIC-COUNT; populate WS-ENTRY(1..N) for all verb entries using
      category headers and two-line verb pairs as described above
   f. GOBACK; `END PROGRAM COBVERBS.`

### Relevant Context
- `cobolinfo/src/cobinfosrv.cblle` lines 1–296 — follow the exact same structure
- Each line is 77 chars: WS-EX (27) + WS-DC (50); keep all text within field limits
- WS-LOAD pattern: set WS-EX and WS-DC, then `MOVE WS-LOAD TO WS-ENTRY(n)`
- ILE COBOL: multiple PROGRAM-ID units in one source file compile together via
  CRTCBLMOD; each PROGRAM-ID is a separate ILE entry point bound into the *SRVPGM

### Status
[x] done

---

## Sub-Task 3 — Expand the array size in cobolinfo.cblle and add topic cycling

### Intent
Make two coordinated changes to `cobolinfo.cblle`:
(a) Grow `WS-TOPIC` from `OCCURS 60 TIMES` to `OCCURS 120 TIMES` to match the
    service program's expanded LINKAGE SECTION.
(b) Add topic-cycling state (a current-topic index) and an F5 handler that
    advances to the next topic, resets the scroll position, and reloads the
    topic data and header before the next display cycle.

### Expected Outcomes
- `cobolinfo.cblle` `WS-TOPIC` `WS-ENTRY OCCURS 60 TIMES` → `OCCURS 120 TIMES`
- New `SPECIAL-NAMES` entry declares the `COBVERBS` procedure:
  `LINKAGE TYPE IS PROCEDURE FOR 'COBVERBS'`
- New working-storage fields:
  - `WS-TOPIC-IDX  PIC 9  COMP-4 VALUE 1` — current topic (1=PIC Clauses, 2=Verbs)
  - `WS-TOPIC-MAX  PIC 9  COMP-4 VALUE 2` — total topics (wraps after this)
- `MAIN-PARA` refactored: calls new `LOAD-TOPIC` paragraph instead of calling
  `COBINFOSRV` directly; `LOAD-TOPIC` dispatches on `WS-TOPIC-IDX` via EVALUATE
- The new `LOAD-TOPIC` paragraph:
  - WHEN 1: CALL `COBINFOSRV` USING BY REFERENCE WS-TOPIC;
            MOVE `'COBOL PICTURE Clauses'` TO WS-HDRTOPIC
  - WHEN 2: CALL `COBVERBS` USING BY REFERENCE WS-TOPIC;
            MOVE `'COBOL Verbs'` TO WS-HDRTOPIC
  - MOVE WS-TOPIC-COUNT TO WS-TOTAL-LINES
- New `WS-IND-F5  PIC X` byte added as byte 4 of `WS-IND-AREA`
- `SCROLL-LOOP` EVALUATE handles `WHEN WS-IND-F5 = '1' PERFORM NEXT-TOPIC`
- New `NEXT-TOPIC` paragraph: increments/wraps `WS-TOPIC-IDX`, resets
  `WS-TOP-LINE` to 1, clears `WS-MSGFLD`, performs LOAD-TOPIC and CALC-PAGES

### Todo List
1. In WORKING-STORAGE, change `WS-ENTRY OCCURS 60 TIMES` → `OCCURS 120 TIMES`
2. Add `LINKAGE TYPE IS PROCEDURE FOR 'COBVERBS'` to `SPECIAL-NAMES`
3. Add `WS-TOPIC-IDX` and `WS-TOPIC-MAX` 77-level items to WORKING-STORAGE
4. Extract the CALL + HDRTOPIC MOVE from `MAIN-PARA` into a new `LOAD-TOPIC`
   paragraph; add EVALUATE dispatch for topics 1 and 2; end with
   MOVE WS-TOPIC-COUNT TO WS-TOTAL-LINES
5. In `MAIN-PARA`, replace the direct CALL with `PERFORM LOAD-TOPIC`;
   keep `PERFORM CALC-PAGES` after it
6. Add `WS-IND-F5  PIC X` as byte 4 of `WS-IND-AREA` (after WS-IND-PGUP)
7. In `SCROLL-LOOP` EVALUATE, add `WHEN WS-IND-F5 = '1' PERFORM NEXT-TOPIC`
8. Write `NEXT-TOPIC` paragraph:
   - Compute: add 1 to WS-TOPIC-IDX; if > WS-TOPIC-MAX move 1 to WS-TOPIC-IDX
   - MOVE 1 TO WS-TOP-LINE
   - MOVE SPACES TO WS-MSGFLD
   - PERFORM LOAD-TOPIC
   - PERFORM CALC-PAGES

### Relevant Context
- `cobolinfo/src/cobolinfo.cblle` lines 32, 56–59, 92–95, 97–107, 128–142
  — SPECIAL-NAMES, indicator area, WS-TOPIC, MAIN-PARA, SCROLL-LOOP
- Indicator byte mapping is determined by DDS keyword declaration order (Sub-Task 4).
  `CA05` is placed after `PAGEUP` in the DDS source so the byte sequence is
  F3=byte1, PgDn=byte2, PgUp=byte3, F5=byte4, matching `WS-IND-AREA` positions.
  This ordering keeps all action-key declarations grouped and makes a future
  `CA06` trivially byte 5 without renumbering.

### Status
[x] done

---

## Sub-Task 4 — Update the display file cobolinfo.dspf

### Intent
Add the `CA05` (F5) key binding to the display file and add the `F5=Next Topic`
legend to the bottom help row.

#### DDS indicator byte-ordering strategy
All action-key keywords are declared together at the **file level** in a
consistent order that maps directly to `WS-IND-AREA` byte positions:

```
CA03(03 'Exit')         → byte 1 of WS-IND-AREA  (WS-IND-F3)
PAGEDOWN(25)            → byte 2                  (WS-IND-PGDN)
PAGEUP(26)              → byte 3                  (WS-IND-PGUP)
CA05(05 'Next Topic')   → byte 4                  (WS-IND-F5)
```

Keeping them grouped and appended in order means a future `CA06` simply adds
byte 5 with no renumbering, and the group is easy to scan at a glance.

### Expected Outcomes
- `CA05(05 'Next Topic')` added at the file-level keyword section, **after**
  `PAGEUP(26)`, so it maps to byte 4 of `WS-IND-AREA`
- Row 23 has an additional static literal `'F5=Next Topic'` after `'PgUp=Scroll Up'`
- All other fields, positions, and attributes are unchanged

### Todo List
1. Append `CA05(05 'Next Topic')` on its own line after `PAGEUP(26)` at the
   file-level keyword section
2. Add `'F5=Next Topic'` static text on row 23 at column 47 (after `PgUp=Scroll Up`
   which ends around col 44)

### Relevant Context
- `cobolinfo/src/cobolinfo.dspf` lines 7–9 — existing CA03/PAGEDOWN/PAGEUP keywords
- `cobolinfo/src/cobolinfo.dspf` lines 53–58 — row 23 legend
- Byte 4 of `WS-IND-AREA` in `cobolinfo.cblle` (`WS-IND-F5`) reads this indicator

### Status
[x] done

---

## Sub-Task 5 — Update the binder language file cobinfobd.bnd

### Intent
Register the new `COBVERBS` export symbol so the service program exposes it and
the main program can resolve it at bind time.

### Expected Outcomes
- `cobinfobd.bnd` has `EXPORT SYMBOL('COBVERBS')` added **after** the existing
  `EXPORT SYMBOL('COBINFOSRV')` line (order matters for signature compatibility)

### Todo List
1. Insert `EXPORT SYMBOL('COBVERBS')` after `EXPORT SYMBOL('COBINFOSRV')` in the
   `STRPGMEXP` block

### Relevant Context
- `cobolinfo/src/cobinfobd.bnd` lines 13–15 — the current STRPGMEXP block
- Comment in the file already states: "Add new PROGRAM-ID names at the END"

### Status
[x] done

---

## Sub-Task 6 — Update README.md

### Intent
Keep the documentation in sync with the code changes.

### Expected Outcomes
- Topics table updated to include `COBVERBS` / `COBOL Verbs` row
- Controls section updated to mention `F5=Next Topic`
- Verb categories listed (matching the entries coded in Sub-Task 1+2)
- Array size note updated (up to 120 entries)

### Todo List
1. Add `COBVERBS` row to the Topics Available table
2. Add `F5=Next Topic` to the Running section controls description
3. Add a "COBOL Verb Categories" section analogous to "PICTURE Clause Categories",
   listing each category name and verb count
4. Update any reference to "60 entries" → "up to 120 entries per topic"

### Relevant Context
- `cobolinfo/README.md` lines 35–69 — Topics and PICTURE Categories sections
- `cobolinfo/README.md` line 69 — "Total: 60 entries displayed 18 per page (4 pages)"
- `cobolinfo/README.md` line 131 — Running / controls description

### Status
[x] done

---

## Implementation Notes

- **Sub-tasks must be done in order: 1+2 → 3 → 4 → 5 → 6.**
  Sub-Tasks 1 and 2 are combined into one atomic edit of `cobinfosrv.cblle`.
  Sub-Task 3 must align its `OCCURS` count with the Sub-Task 1+2 change.
  Sub-Tasks 4 and 5 are prerequisites for a successful build.

- **Indicator byte order in WS-IND-AREA is positional** — bytes are assigned
  strictly by DDS keyword declaration order at the file level.  The chosen
  ordering (CA03, PAGEDOWN, PAGEUP, CA05) maps cleanly to bytes 1–4 and leaves
  byte 5 free for any future action key without renumbering existing positions.

- **ILE COBOL multiple compilation units** — `cobinfosrv.cblle` contains two
  `PROGRAM-ID` sections.  `CRTCBLMOD` compiles both; each PROGRAM-ID becomes a
  separate ILE module entry point and both are bound into `COBINFOSRV *SRVPGM`.

- **Existing COBINFOSRV entries remain unchanged** — the expanded `OCCURS 120`
  does not affect the existing 60-entry fill in COBINFOSRV; only
  `WS-TOPIC-COUNT` controls how many entries the display program reads.
