.PHONY: all
.DELETE_ON_ERROR:
TOPMOD  := UartMod
VLOGFIL := $(TOPMOD).v
VCDFILE := $(TOPMOD).vcd
SIMPROG := $(TOPMOD)_tb
RPTFILE := $(TOPMOD).rpt
BINFILE := $(TOPMOD).bin
SIMFILE := $(SIMPROG).cpp
VDIRFB  := ./obj_dir
PROGPATH := ~/bin/ujprog/ujprog
#COSIMS  := uartsim.cpp
all: $(VCDFILE)

GCC := g++
CFLAGS = -g -Wall -I$(VINC) -I $(VDIRFB)
#
# Modern versions of Verilator and C++ may require an -faligned-new flag
# CFLAGS = -g -Wall -faligned-new -I$(VINC) -I $(VDIRFB)

VERILATOR=verilator
VFLAGS := -O3 -MMD --trace -Wall

## Find the directory containing the Verilog sources.  This is given from
## calling: "verilator -V" and finding the VERILATOR_ROOT output line from
## within it.  From this VERILATOR_ROOT value, we can find all the components
## we need here--in particular, the verilator include directory
VERILATOR_ROOT ?= $(shell bash -c '$(VERILATOR) -V|grep VERILATOR_ROOT | head -1 | sed -e "s/^.*=\s*//"')
##
## The directory containing the verilator includes
VINC := $(VERILATOR_ROOT)/include

$(VDIRFB)/V$(TOPMOD).cpp: $(TOPMOD).v
	$(VERILATOR) $(VFLAGS) -cc $(VLOGFIL)

$(VDIRFB)/V$(TOPMOD)__ALL.a: $(VDIRFB)/V$(TOPMOD).cpp
	make --no-print-directory -C $(VDIRFB) -f V$(TOPMOD).mk

$(SIMPROG): $(SIMFILE) $(VDIRFB)/V$(TOPMOD)__ALL.a $(COSIMS)
	$(GCC) $(CFLAGS) $(VINC)/verilated.cpp				\
		$(VINC)/verilated_vcd_c.cpp $(SIMFILE) $(COSIMS)	\
		$(VDIRFB)/V$(TOPMOD)__ALL.a -o $(SIMPROG)

test: $(VCDFILE)

$(VCDFILE): $(SIMPROG)
	./$(SIMPROG)

##
.PHONY: clean
clean:
	rm -rf $(VDIRFB)/ $(SIMPROG) $(VCDFILE) $(TOPMOD)/ $(BINFILE) $(RPTFILE)
	rm -rf $(TOPMOD).json $(TOPMOD).config $(TOPMOD).bit

##
## Find all of the Verilog dependencies and submodules
##
DEPS := $(wildcard $(VDIRFB)/*.d)

## Include any of these submodules in the Makefile
## ... but only if we are not building the "clean" target
## which would (oops) try to build those dependencies again
##
ifneq ($(MAKECMDGOALS),clean)
ifneq ($(DEPS),)
include $(DEPS)
endif
endif


$(TOPMOD).bit: $(TOPMOD).config
	ecppack $(TOPMOD).config $(TOPMOD).bit

$(TOPMOD).config: $(TOPMOD).json
	nextpnr-ecp5 --85k --package CABGA381 \
        --json $(TOPMOD).json \
		--lpf ulx3s_v20_constraints.lpf \
		--textcfg $(TOPMOD).config

$(TOPMOD).json: $(TOPMOD).ys $(TOPMOD).v
	yosys $(TOPMOD).ys

prog: $(TOPMOD).bit
	sudo $(PROGPATH) $(TOPMOD).bit
