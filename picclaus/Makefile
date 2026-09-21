# ============================================================================
# Makefile for PICCLAUS - ILE COBOL + display file
# ============================================================================
# This Makefile builds two IBM i objects:
#   1. PICCLAUS  - Display file (compiled from DDS)
#   2. PICCLAUS  - ILE COBOL program
#
# Required Parameters:
#   LIB       - Target library for all objects (e.g., LIB=MYLIB)
#
# Optional Parameters:
#   SRCPF     - Source physical file for display file DDS (default: QDDSSRC)
#   SRCDIR    - Source directory (default: src)
#
# Usage Examples:
#   make all LIB=MYLIB                    # Build all objects
#   make all LIB=MYLIB SRCPF=MYSRCPF     # Use custom source file
#   make dspf LIB=MYLIB                   # Build display file only
#   make cobol LIB=MYLIB                  # Build COBOL program only
#   make clean LIB=MYLIB                  # Delete compiled objects from LIB
#   make help                             # Show this help
# ============================================================================

SHELL = /QOpenSys/pkgs/bin/bash

# Default values for optional parameters
SRCPF  ?= QDDSSRC
SRCDIR ?= src

# Source files
DSPF_SRC = $(SRCDIR)/picclaus.dspf
COBL_SRC = $(SRCDIR)/picclaus.cblle

# Member names
DSPF_MBR = PICCLAUS
COBL_MBR = PICCLAUS

# IBM i command wrapper
SYSTEM = system

.PHONY: all dspf cobol clean help check-params

# Default target
all: check-params dspf cobol
	@echo "============================================"
	@echo "Build complete!"
	@echo "Objects created in library: $(LIB)"
	@echo "  - $(LIB)/$(DSPF_MBR) (Display File)"
	@echo "  - $(LIB)/$(COBL_MBR) (ILE COBOL Program)"
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
	    TEXT('PICCLAUS display file') REPLACE(*YES)"
	@echo "Display file $(DSPF_MBR) created successfully in $(LIB)"
	@echo ""

# -----------------------------------------------------------------------
# ILE COBOL program
#   Compiled directly from the IFS stream file.
#   The display file must already exist in LIB (built by the dspf target).
# -----------------------------------------------------------------------
cobol: check-params dspf
	@echo "Building ILE COBOL program $(COBL_MBR)..."
	$(SYSTEM) "CRTBNDCBL PGM($(LIB)/$(COBL_MBR)) \
	    SRCSTMF('$(CURDIR)/$(COBL_SRC)') \
	    DBGVIEW(*SOURCE) REPLACE(*YES)" \
		TGTCCSID(*JOB)
	@echo "ILE COBOL program $(COBL_MBR) created successfully in $(LIB)"
	@echo ""

# -----------------------------------------------------------------------
# Remove compiled objects from the target library
# -----------------------------------------------------------------------
clean: check-params
	-$(SYSTEM) "DLTOBJ OBJ($(LIB)/$(DSPF_MBR)) OBJTYPE(*FILE)"
	-$(SYSTEM) "DLTOBJ OBJ($(LIB)/$(COBL_MBR)) OBJTYPE(*PGM)"

# Help target
help:
	@echo "============================================"
	@echo "PICCLAUS Makefile - Build Instructions"
	@echo "============================================"
	@echo ""
	@echo "This Makefile builds two IBM i objects:"
	@echo "  1. $(DSPF_MBR) (Display File)"
	@echo "  2. $(COBL_MBR) (ILE COBOL Program)"
	@echo ""
	@echo "Required Parameters:"
	@echo "  LIB       - Target library for all objects"
	@echo ""
	@echo "Optional Parameters:"
	@echo "  SRCPF     - Source physical file for DDS (default: QDDSSRC)"
	@echo "  SRCDIR    - Source directory (default: src)"
	@echo ""
	@echo "Targets:"
	@echo "  all    - Build display file and COBOL program (default)"
	@echo "  dspf   - Build display file only"
	@echo "  cobol  - Build COBOL program (includes dspf)"
	@echo "  clean  - Delete compiled objects from LIB"
	@echo "  help   - Show this help message"
	@echo ""
	@echo "Usage Examples:"
	@echo "  make all LIB=MYLIB"
	@echo "  make all LIB=MYLIB SRCPF=MYSRCPF"
	@echo "  make dspf LIB=MYLIB"
	@echo "  make cobol LIB=MYLIB"
	@echo "  make clean LIB=MYLIB"
	@echo ""
	@echo "Prerequisites:"
	@echo "  - Source files must be in SRCDIR (default: src/)"
	@echo "  - Target library (LIB) must exist on IBM i"
	@echo ""
	@echo "============================================"

# Made with Bob
