# ============================================================================
# Makefile for COBOLINFO - ILE COBOL Reference program
# ============================================================================
# This Makefile builds four IBM i objects:
#   1. COBINFODSP  - Display file (compiled from DDS)
#   2. COBINFOSRV  - ILE COBOL service program (content layer)
#   3. COBINFOBD   - Binding directory
#   4. COBOLINFO   - ILE COBOL main program (display layer)
#
# Required Parameters:
#   LIB       - Target library for all objects (e.g., LIB=MYLIB)
#
# Optional Parameters:
#   SRCPF     - Source physical file for display file DDS (default: QDDSSRC)
#   BNDSRCPF  - Source physical file for binder language (default: QBNDSRC)
#   SRCDIR    - Source directory (default: src)
#
# Usage Examples:
#   make all LIB=MYLIB                    # Build all objects
#   make all LIB=MYLIB SRCPF=MYSRCPF     # Use custom DDS source file
#   make all LIB=MYLIB BNDSRCPF=MYBNDSRC # Use custom binder source file
#   make dspf LIB=MYLIB                   # Build display file only
#   make srvpgm LIB=MYLIB                 # Build service program and bnddir
#   make cobol LIB=MYLIB                  # Build main COBOL program only
#   make clean LIB=MYLIB                  # Delete compiled objects from LIB
#   make help                             # Show this help
# ============================================================================

SHELL = /QOpenSys/pkgs/bin/bash

# Default values for optional parameters
SRCPF    ?= QDDSSRC
BNDSRCPF ?= QBNDSRC
SRCDIR   ?= src

# Source files
DSPF_SRC   = $(SRCDIR)/cobolinfo.dspf
SRVPGM_SRC = $(SRCDIR)/cobinfosrv.cblle
BND_SRC    = $(SRCDIR)/cobinfobd.bnd
COBL_SRC   = $(SRCDIR)/cobolinfo.cblle

# Object / member names
DSPF_MBR   = COBINFODSP
SRVPGM_MBR = COBINFOSRV
BNDDIR_MBR = COBINFOBD
COBL_MBR   = COBOLINFO

# IBM i command wrapper
SYSTEM = system

.PHONY: all dspf srvpgm cobol clean help check-params

# Default target
all: check-params dspf srvpgm cobol
	@echo "============================================"
	@echo "Build complete!"
	@echo "Objects created in library: $(LIB)"
	@echo "  - $(LIB)/$(DSPF_MBR)   (Display File)"
	@echo "  - $(LIB)/$(SRVPGM_MBR) (ILE COBOL Service Program)"
	@echo "  - $(LIB)/$(BNDDIR_MBR) (Binding Directory)"
	@echo "  - $(LIB)/$(COBL_MBR)   (ILE COBOL Program)"
	@echo "============================================"

# Check required parameters
check-params:
ifndef LIB
	@echo "============================================"
	@echo "ERROR: Required parameter LIB is not set"
	@echo "============================================"
	@echo ""
	@echo "Usage:"
	@echo "  make all LIB=<library> [SRCPF=<srcpf>] [SRCDIR=<srcdir>]"
	@echo ""
	@echo "Required Parameters:"
	@echo "  LIB       - Target library for all objects"
	@echo ""
	@echo "Optional Parameters:"
	@echo "  SRCPF     - Source physical file for DDS (default: QDDSSRC)"
	@echo "  SRCDIR    - Source directory (default: src)"
	@echo ""
	@echo "Examples:"
	@echo "  make all LIB=MYLIB"
	@echo "  make all LIB=MYLIB SRCPF=MYSRCPF"
	@echo "  make dspf LIB=MYLIB"
	@echo "  make srvpgm LIB=MYLIB"
	@echo "  make cobol LIB=MYLIB"
	@echo ""
	@echo "============================================"
	@exit 1
endif
	@echo "Building with parameters:"
	@echo "  LIB    = $(LIB)"
	@echo "  SRCPF  = $(SRCPF)"
	@echo "  SRCDIR = $(SRCDIR)"
	@echo "============================================"

# -----------------------------------------------------------------------
# Display file
#   1. Create QDDSSRC in the target library if it does not exist;
#      the `-` prefix ignores CPF7302 (file already exists).
#   2. Copy the DDS source from the IFS into the source member.
#   3. Compile the display file object into the target library.
# -----------------------------------------------------------------------
dspf: check-params
	@echo "Building display file $(DSPF_MBR)..."
	@echo "Step 1: Creating source file $(LIB)/$(SRCPF) (if needed)..."
	-$(SYSTEM) "CRTSRCPF FILE($(LIB)/$(SRCPF)) RCDLEN(112) TEXT('DDS source')"
	@echo "Step 2: Copying $(DSPF_SRC) to source member..."
	$(SYSTEM) "CPYFRMSTMF FROMSTMF('$(CURDIR)/$(DSPF_SRC)') \
	    TOMBR('/QSYS.LIB/$(LIB).LIB/$(SRCPF).FILE/$(DSPF_MBR).MBR') \
	    MBROPT(*REPLACE) STMFCCSID(*STMF) DBFCCSID(*FILE)"
	@echo "Step 3: Compiling display file..."
	$(SYSTEM) "CRTDSPF FILE($(LIB)/$(DSPF_MBR)) \
	    SRCFILE($(LIB)/$(SRCPF)) SRCMBR($(DSPF_MBR)) \
	    TEXT('COBOLINFO display file') REPLACE(*YES) AUT(*EXCLUDE)"
	@echo "Display file $(DSPF_MBR) created successfully in $(LIB)"
	@echo ""

# -----------------------------------------------------------------------
# Service program
#   1. Compile cobinfosrv.cblle to a *MODULE (CRTCBLMOD).
#   2. Stage the binder language source into a source physical file member
#      (CPYFRMSTMF handles the UTF-8 -> EBCDIC CCSID conversion automatically,
#      the same way the DDS source is staged).
#   3. Create the service program using the staged source member.
#   4. Create the binding directory COBINFOBD (ignored if it already exists).
#   5. Add COBINFOSRV *SRVPGM to the binding directory (ignored if already present).
#
# NOTE: Steps 4-5 use single-line system calls so that the make `-` error-ignore
# prefix covers the entire command (it does not propagate across line continuations).
# The *SRVPGM must exist (step 3) before it can be added to the bnddir (step 5).
# -----------------------------------------------------------------------
BND_MBR = COBINFOBD

srvpgm: check-params
	@echo "Building service program $(SRVPGM_MBR)..."
	@echo "Step 1: Compiling service program module..."
	$(SYSTEM) "CRTCBLMOD MODULE($(LIB)/$(SRVPGM_MBR)) SRCSTMF('$(CURDIR)/$(SRVPGM_SRC)') DBGVIEW(*SOURCE) OPTION(*APOST) TGTCCSID(*JOB) REPLACE(*YES) AUT(*EXCLUDE)"
	@echo "Step 2: Copying binder language source to $(LIB)/$(BNDSRCPF)($(BND_MBR))..."
	-$(SYSTEM) "CRTSRCPF FILE($(LIB)/$(BNDSRCPF)) RCDLEN(112) TEXT('Binder language source')"
	$(SYSTEM) "CPYFRMSTMF FROMSTMF('$(CURDIR)/$(BND_SRC)') TOMBR('/QSYS.LIB/$(LIB).LIB/$(BNDSRCPF).FILE/$(BND_MBR).MBR') MBROPT(*REPLACE) STMFCCSID(*STMF) DBFCCSID(*FILE)"
	@echo "Step 3: Creating service program with binder language..."
	$(SYSTEM) "CRTSRVPGM SRVPGM($(LIB)/$(SRVPGM_MBR)) MODULE($(LIB)/$(SRVPGM_MBR) $(LIB)/COBVERBS) EXPORT(*SRCFILE) SRCFILE($(LIB)/$(BNDSRCPF)) SRCMBR($(BND_MBR)) ACTGRP(*CALLER) TEXT('COBOLINFO content service program') REPLACE(*YES) AUT(*EXCLUDE)"
	@echo "Step 4: Creating binding directory $(BNDDIR_MBR) (if needed)..."
	-$(SYSTEM) "CRTBNDDIR BNDDIR($(LIB)/$(BNDDIR_MBR)) TEXT('COBOLINFO binding directory') AUT(*EXCLUDE)"
	@echo "Step 5: Adding $(SRVPGM_MBR) *SRVPGM to binding directory..."
	-$(SYSTEM) "ADDBNDDIRE BNDDIR($(LIB)/$(BNDDIR_MBR)) OBJ(($(LIB)/$(SRVPGM_MBR) *SRVPGM))"
	@echo "Service program $(SRVPGM_MBR) created successfully in $(LIB)"
	@echo ""

# -----------------------------------------------------------------------
# ILE COBOL main program
#   Compiled directly from the IFS stream file.
#   The display file and service program must already exist in LIB.
#   All three CL operations run in one shell invocation (single recipe line)
#   so that ADDLIBLE, CRTBNDCBL, and RMVLIBLE share the same job context.
#   `cl` is the bash builtin for in-process CL; `system` spawns a new job.
#   Make spawns a new subshell per recipe line, so splitting them would
#   lose the library list change before CRTBNDCBL runs.
# -----------------------------------------------------------------------
cobol: check-params dspf srvpgm
	@echo "Building ILE COBOL program $(COBL_MBR)..."
	cl "ADDLIBLE LIB($(LIB)) POSITION(*FIRST)" ; $(SYSTEM) "CRTBNDCBL PGM($(LIB)/$(COBL_MBR)) SRCSTMF('$(CURDIR)/$(COBL_SRC)') BNDDIR($(LIB)/$(BNDDIR_MBR)) DBGVIEW(*SOURCE) REPLACE(*YES) OPTION(*APOST) TGTCCSID(*JOB) AUT(*EXCLUDE)" ; cl "RMVLIBLE LIB($(LIB))"
	@echo "ILE COBOL program $(COBL_MBR) created successfully in $(LIB)"
	@echo ""

# -----------------------------------------------------------------------
# Remove compiled objects from the target library
# -----------------------------------------------------------------------
clean: check-params
	-$(SYSTEM) "DLTOBJ OBJ($(LIB)/$(COBL_MBR))   OBJTYPE(*PGM)"
	-$(SYSTEM) "DLTOBJ OBJ($(LIB)/$(SRVPGM_MBR)) OBJTYPE(*SRVPGM)"
	-$(SYSTEM) "DLTOBJ OBJ($(LIB)/$(SRVPGM_MBR)) OBJTYPE(*MODULE)"
	-$(SYSTEM) "DLTOBJ OBJ($(LIB)/COBVERBS)       OBJTYPE(*MODULE)"
	-$(SYSTEM) "DLTOBJ OBJ($(LIB)/$(BNDDIR_MBR)) OBJTYPE(*BNDDIR)"
	-$(SYSTEM) "DLTOBJ OBJ($(LIB)/$(DSPF_MBR))   OBJTYPE(*FILE)"

# Help target
help:
	@echo "============================================"
	@echo "COBOLINFO Makefile - Build Instructions"
	@echo "============================================"
	@echo ""
	@echo "This Makefile builds four IBM i objects:"
	@echo "  1. $(DSPF_MBR)   (Display File)"
	@echo "  2. $(SRVPGM_MBR) (ILE COBOL Service Program - content)"
	@echo "  3. $(BNDDIR_MBR) (Binding Directory)"
	@echo "  4. $(COBL_MBR)   (ILE COBOL Program - display)"
	@echo ""
	@echo "Required Parameters:"
	@echo "  LIB       - Target library for all objects"
	@echo ""
	@echo "Optional Parameters:"
	@echo "  SRCPF     - Source physical file for DDS (default: QDDSSRC)"
	@echo "  SRCDIR    - Source directory (default: src)"
	@echo ""
	@echo "Targets:"
	@echo "  all     - Build all objects (default)"
	@echo "  dspf    - Build display file only"
	@echo "  srvpgm  - Build service program and binding directory"
	@echo "  cobol   - Build main COBOL program (includes dspf + srvpgm)"
	@echo "  clean   - Delete compiled objects from LIB"
	@echo "  help    - Show this help message"
	@echo ""
	@echo "Usage Examples:"
	@echo "  make all LIB=MYLIB"
	@echo "  make all LIB=MYLIB SRCPF=MYSRCPF"
	@echo "  make all LIB=MYLIB BNDSRCPF=MYBNDSRC"
	@echo "  make srvpgm LIB=MYLIB"
	@echo "  make cobol LIB=MYLIB"
	@echo "  make clean LIB=MYLIB"
	@echo ""
	@echo "Prerequisites:"
	@echo "  - Source files must be in SRCDIR (default: src/)"
	@echo "  - Target library (LIB) must exist on IBM i"
	@echo "  - ILE COBOL (5770-CB1) must be installed"
	@echo ""
	@echo "============================================"

# Made with Bob
