# Use MAIN_MAKEFILE=1 for compatibility
MAIN_MAKEFILE=1
include ffbuild/config.mak

SRC_DIR = $(SRC_PATH_BARE)

vpath %.c   $(SRC_DIR)
vpath %.h   $(SRC_DIR)
vpath %.S   $(SRC_DIR)
vpath %.asm $(SRC_DIR)
vpath %.v   $(SRC_DIR)
vpath %.texi $(SRC_PATH_BARE)

# Cross compile toolchain variables (usually from config.mak)
CC ?= aarch64-linux-gnu-gcc
LD ?= aarch64-linux-gnu-ld
AR ?= aarch64-linux-gnu-ar
AS ?= aarch64-linux-gnu-as
NM ?= aarch64-linux-gnu-nm
STRIP ?= aarch64-linux-gnu-strip

PROGS-$(CONFIG_FFMPEG)   += ffmpeg
PROGS-$(CONFIG_FFPLAY)   += ffplay
PROGS-$(CONFIG_FFPROBE)  += ffprobe
PROGS-$(CONFIG_FFSERVER) += ffserver

PROGS      := $(PROGS-yes:%=%$(EXESUF))
PROGS_G     = $(PROGS-yes:%=%_g$(EXESUF))
OBJS        = $(PROGS-yes:%=%.o) cmdutils.o
MANPAGES    = $(PROGS-yes:%=doc/%.1)
PODPAGES    = $(PROGS-yes:%=doc/%.pod)
HTMLPAGES   = $(PROGS-yes:%=doc/%.html)
TOOLS       = $(addprefix tools/, $(addsuffix $(EXESUF), cws2fws graph2dot lavfi-showfiltfmts pktdumper probetest qt-faststart trasher))
TESTTOOLS   = audiogen videogen rotozoom tiny_psnr base64
HOSTPROGS  := $(TESTTOOLS:%=tests/%)

BASENAMES   = ffmpeg ffplay ffprobe ffserver
ALLPROGS    = $(BASENAMES:%=%$(EXESUF))
ALLPROGS_G  = $(BASENAMES:%=%_g$(EXESUF))
ALLMANPAGES = $(BASENAMES:%=%.1)

ALLFFLIBS = avcodec avdevice avfilter avformat avutil postproc swscale

FFLIBS-$(CONFIG_AVDEVICE) += avdevice
FFLIBS-$(CONFIG_AVFILTER) += avfilter
FFLIBS-$(CONFIG_AVFORMAT) += avformat
FFLIBS-$(CONFIG_AVCODEC)  += avcodec
FFLIBS-$(CONFIG_POSTPROC) += postproc
FFLIBS-$(CONFIG_SWSCALE)  += swscale

FFLIBS := avutil

DATA_FILES := $(wildcard $(SRC_DIR)/ffpresets/*.ffpreset)

SKIPHEADERS = cmdutils_common_opts.h

include common.mak

FF_LDFLAGS   := $(FFLDFLAGS)
FF_EXTRALIBS := $(FFEXTRALIBS)
FF_DEP_LIBS  := $(DEP_LIBS)

all-$(CONFIG_DOC): documentation

all: $(FF_DEP_LIBS) $(PROGS)

$(PROGS): %$(EXESUF): %_g$(EXESUF)
	$(CP) $< $@
	$(STRIP) $@

config.h: .config
.config: $(wildcard $(FFLIBS:%=$(SRC_DIR)/lib%/all*.c))
	@-tput bold 2>/dev/null
	@-printf '\nWARNING: $(?F) newer than config.h, rerun configure\n\n'
	@-tput sgr0 2>/dev/null

SUBDIR_VARS := OBJS FFLIBS CLEANFILES DIRS TESTPROGS EXAMPLES SKIPHEADERS \
               ALTIVEC-OBJS MMX-OBJS NEON-OBJS X86-OBJS YASM-OBJS-FFT YASM-OBJS \
               HOSTPROGS BUILT_HEADERS TESTOBJS ARCH_HEADERS ARMV6-OBJS

define RESET
$(1) :=
$(1)-yes :=
endef

define DOSUBDIR
$(foreach V,$(SUBDIR_VARS),$(eval $(call RESET,$(V))))
SUBDIR := $(1)/
include $(1)/Makefile
endef

$(foreach D,$(FFLIBS),$(eval $(call DOSUBDIR,lib$(D))))

ffplay.o: CFLAGS += $(SDL_CFLAGS)
ffplay_g$(EXESUF): FF_EXTRALIBS += $(SDL_LIBS)
ffserver_g$(EXESUF): FF_LDFLAGS += $(FFSERVERLDFLAGS)

%_g$(EXESUF): %.o cmdutils.o $(FF_DEP_LIBS)
	$(LD) $(FF_LDFLAGS) -o $@ $< cmdutils.o $(FF_EXTRALIBS)

alltools: $(TOOLS)

tools/%$(EXESUF): tools/%.o
	$(LD) $(FF_LDFLAGS) -o $@ $< $(FF_EXTRALIBS)

tools/%.o: tools/%.c
	$(CC) $(CPPFLAGS) $(CFLAGS) -c $(CC_O) $<

-include $(wildcard tools/*.d)
-include $(wildcard tests/*.d)
