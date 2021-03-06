PIN=../../pin-2.13
BASE_INCLUDE_DIR=./../include

SUFFIXES += .d

CXX=g++
OPTIMIZATION=-O3
CXXFLAGS_BASE=-std=c++11 -Wall -pipe -D_FILE_OFFSET_BITS=64
CXXFLAGS_PIN=-fomit-frame-pointer \
             -DBIGARRAY_MULTIPLIER=1 \
             -DUSING_XED \
             -fno-stack-protector \
             -fno-strict-aliasing \
             -fomit-frame-pointer \
             -fPIC \
             -DTARGET_LINUX \
             -DTARGET_IA32E -DHOST_IA32E # 64 bit stuffz
             #-DTARGET_IA32 -DHOST_IA32 # 32 bit stuffz

CXXFLAGS_FORENSIX=-DFORENSIX_SLICING -DHEAP_DUMP_AT_SLICE_OUTPUT

INCFLAGS_BASE=-I. -I$(BASE_INCLUDE_DIR)
INCFLAGS_PIN=-I${PIN}/source/tools/include \
             -I${PIN}/source/tools/InstLib \
             -I${PIN}/extras/xed2-intel64/include \
             -I${PIN}/extras/components/include \
             -I${PIN}/source/include/pin \
             -I${PIN}/source/include/pin/gen
INCFLAGS_FORENSIX=-I./models

LDFLAGS_PIN=-shared \
            -Wl,--hash-style=sysv \
            -Wl,-Bsymbolic \
            -Wl,--version-script=${PIN}/source/include/pin/pintool.ver \
            -L${PIN}/extras/xed2-intel64/lib \
            -L${PIN}/intel64/lib \
            -L${PIN}/intel64/lib-ext

LIBS+=-lpin -lxed -ldwarf -lelf

CXXFLAGS_LIBFORENSIX=$(CXXFLAGS_BASE) \
                     $(CXXFLAGS_FORENSIX) \
                     $(CXXFLAGS_PIN) \
                     $(INCFLAGS_BASE) \
                     $(INCFLAGS_PIN) \
                     $(INCFLAGS_FORENSIX)


CXXFLAGS+=$(CXXFLAGS_LIBFORENSIX)
LDFLAGS+=$(LDFLAGS_PIN) -Wall $(OPTIMIZATION)

SRC_DIR=.
SRCS=$(shell find $(SRC_DIR) -name "*.cpp")
OBJS=$(SRCS:%.cpp=%.o)

INCLUDE_SRCS=$(shell find $(BASE_INCLUDE_DIR) -name "*.cpp")
INCLUDE_OBJS=$(INCLUDE_SRCS:%.cpp=%.o)

OBJS+=$(INCLUDE_OBJS)
DO_NOT_TIDY=./pin_models_gtk_helper.o

GOALS=libforensixslicing.so

all:$(GOALS)

tidy:
	$(MAKE) tidy_includes
	rm -rf $(filter-out $(DO_NOT_TIDY), $(OBJS))

clean:
	$(MAKE) tidy
	rm -rf $(GOALS) *.d $(OBJS)
	rm *.P

tidy_includes:
	rm -rf $(INCLUDE_OBJS)

# Linker command
$(GOALS):$(OBJS)
	$(CXX) $(LDFLAGS) -o $@ $^ $(LIBS)
	$(MAKE) tidy_includes

#Compilation
pin_models.o: pin_models.cpp
	$(CXX) $(CXXFLAGS) -MD -c -O0 -o $@ $<
	@cp $*.d $*.P; \
     sed -e 's/#.*//' -e 's/^[^:]*: *//' -e 's/ *\\$$//' \
         -e '/^$$/ d' -e 's/$$/ :/' < $*.d >> $*.P; \
     rm -f $*.d

%.o: %.cpp
	$(CXX) $(CXXFLAGS) -MD -c $(OPTIMIZATION) -o $@ $<
	@cp $*.d $*.P; \
     sed -e 's/#.*//' -e 's/^[^:]*: *//' -e 's/ *\\$$//' \
         -e '/^$$/ d' -e 's/$$/ :/' < $*.d >> $*.P; \
     rm -f $*.d

-include *.P
