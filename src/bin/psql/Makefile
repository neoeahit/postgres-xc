#-------------------------------------------------------------------------
#
# Makefile for src/bin/psql
#
# Portions Copyright (c) 1996-2012, PostgreSQL Global Development Group
# Portions Copyright (c) 1994, Regents of the University of California
#
# src/bin/psql/Makefile
#
#-------------------------------------------------------------------------

PGFILEDESC = "psql - the PostgreSQL interactive terminal"
PGAPPICON=win32

subdir = src/bin/psql
top_builddir = ../../..
include $(top_builddir)/src/Makefile.global

REFDOCDIR= $(top_srcdir)/doc-xc/src/sgml/ref

MAKESGMLDIR = $(top_builddir)/src/pgxc/tools/makesgml
SGMLDIR= $(top_builddir)/doc-xc/src/sgml

override CPPFLAGS := -I. -I$(srcdir) -I$(libpq_srcdir) -I$(top_srcdir)/src/bin/pg_dump $(CPPFLAGS)

OBJS=	command.o common.o help.o input.o stringutils.o mainloop.o copy.o \
	startup.o prompt.o variables.o large_obj.o print.o describe.o \
	tab-complete.o mbprint.o dumputils.o keywords.o kwlookup.o \
	sql_help.o \
	$(WIN32RES)

FLEXFLAGS = -Cfe -b -p -p


all: psql

psql: $(OBJS) | submake-libpq submake-libpgport
	$(CC) $(CFLAGS) $(OBJS) $(libpq_pgport) $(LDFLAGS) $(LDFLAGS_EX) $(LIBS) -o $@$(X)

help.o: sql_help.h

dumputils.c keywords.c: % : $(top_srcdir)/src/bin/pg_dump/%
	rm -f $@ && $(LN_S) $< .

kwlookup.c: % : $(top_srcdir)/src/backend/parser/%
	rm -f $@ && $(LN_S) $< .

sql_help.c: sql_help.h ;
sql_help.h: create_help.pl $(wildcard $(REFDOCDIR)/*.sgml)
	$(MAKE) -C $(MAKESGMLDIR)
	$(MAKE) -C $(SGMLDIR) sgml-files
	$(PERL) $< $(REFDOCDIR) $*

# psqlscan is compiled as part of mainloop
mainloop.o: psqlscan.c

psqlscan.c: psqlscan.l
ifdef FLEX
	$(FLEX) $(FLEXFLAGS) -o'$@' $<
	@if [ `wc -l <lex.backup` -eq 1 ]; then rm lex.backup; else echo "Scanner requires backup, see lex.backup."; exit 1; fi
else
	@$(missing) flex $< $@
endif

distprep: sql_help.h psqlscan.c

install: all installdirs
	$(INSTALL_PROGRAM) psql$(X) '$(DESTDIR)$(bindir)/psql$(X)'
	$(INSTALL_DATA) $(srcdir)/psqlrc.sample '$(DESTDIR)$(datadir)/psqlrc.sample'

installdirs:
	$(MKDIR_P) '$(DESTDIR)$(bindir)'

uninstall:
	rm -f '$(DESTDIR)$(bindir)/psql$(X)' '$(DESTDIR)$(datadir)/psqlrc.sample'

# psqlscan.c is in the distribution tarball, so is not cleaned here
clean distclean:
	rm -f psql$(X) $(OBJS) dumputils.c keywords.c kwlookup.c lex.backup

maintainer-clean: distclean
	rm -f sql_help.h sql_help.c psqlscan.c
