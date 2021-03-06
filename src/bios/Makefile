.SUFFIXES: .cc

srcdir = .

ROMBIOS = rombios-de1.c

SHELL = /bin/sh
CXX = g++
CXXFLAGS = -g3 -O0 -D_FILE_OFFSET_BITS=64 -D_LARGE_FILES
CC      = gcc
CFLAGS  = -g -O2 -Wall -Wstrict-prototypes

LDFLAGS = 
LIBS =  -lm
RANLIB = ranlib

BCC = bcc
GCC = gcc
GCC32 = gcc -m32
AS86 = as86

BX_INCDIRS = -I.. -I$(srcdir)/.. -I../iodev -I$(srcdir)/../iodev
LOCAL_CXXFLAGS =

#
# -------- end configurable options --------------------------
#

RELEASE = `pwd | sed "s-.*/--"`
BUILDDATE = `date '+%d %b %Y'`
BUILD_DATE = `date '+%m/%d/%y'`
RELDATE = `git log -1 | grep Date | sed 's/Date:   //g'`
RELVERS = `git describe`

BIOS_BUILD_DATE = "-DBIOS_BUILD_DATE=\"$(BUILD_DATE)\""
BIOS_BUILDDATE = "-DBIOS_BUILDDATE=\"$(BUILDDATE)\""
BIOS_DATE = "-DBIOS_DATE=\"$(RELDATE)\""
BIOS_VERS = "-DBIOS_VERS=\"$(RELVERS)\""

all: de1
de1: bios.out

../../bin/ff: ../tools/ff.c
	$(CC) ../tools/ff.c -o ../tools/ff
	cp ../tools/ff ../../bin/ff

ff.out: ../../bin/ff
	../../bin/ff 8192 >$@

%.ml403: %.rom
	hexdump -v -e '4/1 "%02X"' -e '"\n"' $< > ../../impl/virtex4-ml403ep/sim/$@

%.rom: %.out
	../../bin/out2rom-ml403 ff.out $< >$@

#vgabios.rom: vgabios.out
#	out2rom-ml403 ff.out $< >$@

#hd.rom: hd.out
#	out2rom-ml403 ff.out $< >$@

bios.out: vgabios.out ff.out zet-bios.out
	cat vgabios.out ff.out zet-bios.out > $@

run: flash.rom
	prf

clean:
	rm -f  *.o *.a *.s _rombios*_.c rombios*.txt rombios*.sym *.out *.rom
	rm -f  usage biossums vgasums
	rm -f  *.ld86 \
          temp.awk.* vgabios*.orig _vgabios_* vgabios*.bin vgabios*.txt $(RELEASE).bin *.bak

bios-clean:
	rm -f  zet-bios.out

.cc.o:
	$(CXX) -c $(BX_INCDIRS) $(CXXFLAGS) $(LOCAL_CXXFLAGS) $< -o $@

zet-bios.out: $(ROMBIOS) biossums rombios.h
	$(GCC32) $(BIOS_VERS) $(BIOS_DATE) $(BIOS_BUILD_DATE) $(BIOS_BUILDDATE) -DLEGACY -E -P $< > _rombiosl_.c
	$(BCC) -o rombiosl.s -C-c -D__i86__ -0 -S _rombiosl_.c
	sed -e 's/^\.text//' -e 's/^\.data//' rombiosl.s > _rombiosl_.s
	$(AS86) _rombiosl_.s -b tmpl.bin -u- -w- -g -0 -j -O -l rombiosl.txt
	-perl ${srcdir}/makesym.perl < rombiosl.txt > rombiosl.sym
	mv tmpl.bin $@
	./biossums $@
	rm -f  _rombiosl_.s

vgabios.out: vgabios.c vgabios.h vgatables.h vgasums
	$(GCC) -E vgabios.c $(BIOS_VERS) -P $(BIOS_BUILDDATE) > _vgabios_.c
	$(BCC) -o vgabios.s -C-c -D__i86__ -S -0 _vgabios_.c
	sed -e 's/^\.text//' -e 's/^\.data//' vgabios.s > _vgabios_.s
	$(AS86) _vgabios_.s -b vgabios.bin -u -w- -g -0 -j -O -l vgabios.txt
#	rm -f _vgabios_.s _vgabios_.c vgabios.s
	mv vgabios.bin $@
	./vgasums $@

%.rtlrom: %.out
	hexdump -v -e '1/1 "%02X"' -e '"\n"' $< > ../../sim/$@

vgasums: vgasums.c
	$(CC) -o vgasums vgasums.c

biossums: biossums.c
	$(GCC) -o biossums biossums.c
