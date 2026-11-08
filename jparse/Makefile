OBJLIB ?= MYLIB
SRCDIR ?= /home/MYUSER/builds/ibmi-cobol-examples/jparse

.PHONY: all jparsei

all: jparsei

jparsei:
	cl "CRTSQLCBLI OBJ($(OBJLIB)/JPARSEI) SRCSTMF('$(SRCDIR)/jparsei.sqlcblle') OPTION(*APOST) CVTCCSID(*JOB)"
