.PHONY: csim cosim hw hls clean host

APP ?= workload
SDA_VER ?= 2016.4

CSIM_XCLBIN ?= $(APP)-csim.xclbin
COSIM_XCLBIN ?= $(APP)-cosim.xclbin
HW_XCLBIN ?= $(APP)-hw.xclbin

KERNEL_SRCS ?= aes.cpp
KERNEL_NAME ?= $(APP)
HOST_SRCS ?= local_support.cpp support.cpp harness.cpp
HOST_ARGS ?= input.data check.data
HOST_BIN ?= $(APP)

SRC ?= .
OBJ ?= obj
BIN ?= bin
BIT ?= bit
RPT ?= rpt

CXX ?= g++
CLCXX ?= xocc

ifeq ($(SDA_VER),2016.2)
	XILINX_SDACCEL ?= /opt/tools/SDAccel_$(SDA_VER)
else
	XILINX_SDACCEL ?= /opt/tools/xilinx/SDx/$(SDA_VER)
endif
WITH_SDACCEL = SDA_VER=$(SDA_VER) with-sdaccel

HOST_CFLAGS = -std=c++0x -g -Wall -DFPGA_DEVICE -DC_KERNEL -I$(XILINX_SDACCEL)/runtime/include/1_2 -I/curr/pengwei/dse/HDLRevisit/AlphaData_Optimization/common
HOST_LFLAGS = -L$(XILINX_SDACCEL)/runtime/lib/x86_64 -lxilinxopencl -lrt

XDEVICE ?= xilinx:adm-pcie-7v3:1ddr:3.0
ifeq ($(XDEVICE),xilinx:aws-vu9p-f1:4ddr-xpr-2pr:4.0)
	SDA_VER = 2017.1
endif
ifeq ($(SDA_VER),2017.2)
	XILINX_SDX ?= /opt/tools/xilinx/SDx/$(SDA_VER)
	HOST_CFLAGS += -DTARGET_DEVICE=\"$(subst :,_,$(subst .,_,$(XDEVICE)))\"
else
	HOST_CFLAGS += -DTARGET_DEVICE=\"$(XDEVICE)\"
endif

CLCXX_OPT = $(CLCXX_OPT_LEVEL) $(DEVICE_REPO_OPT) --platform $(XDEVICE) $(KERNEL_DEFS) $(KERNEL_INCS)
CLCXX_OPT += --kernel $(KERNEL_NAME)
CLCXX_OPT += -s -g
CLCXX_CSIM_OPT = -t sw_emu
CLCXX_COSIM_OPT = -t hw_emu
CLCXX_HW_OPT = -t hw

csim: $(BIN)/$(HOST_BIN) $(BIT)/$(CSIM_XCLBIN)
	XCL_EMULATION_MODE=true $(WITH_SDACCEL) $^ $(HOST_ARGS)

cosim: $(BIN)/$(HOST_BIN) $(BIT)/$(COSIM_XCLBIN)
	XCL_EMULATION_MODE=true $(WITH_SDACCEL) $^ $(HOST_ARGS)

hw: $(BIN)/$(HOST_BIN) $(BIT)/$(HW_XCLBIN)

hls: $(OBJ)/$(HW_XCLBIN:.xclbin=.xo)

host: $(BIN)/$(HOST_BIN)

clean:
	rm -rf $(BIN) $(BIT) $(RPT) $(OBJ) _xocc_* .Xil sdaccel_profile_summary.{csv,html}

$(BIN)/$(HOST_BIN): $(HOST_SRCS:%.cpp=$(OBJ)/%.o)
	@mkdir -p $(BIN)
	$(WITH_SDACCEL) $(CXX) $(HOST_LFLAGS) $^ -o $@

$(OBJ)/%.o: $(SRC)/%.cpp
	@mkdir -p $(OBJ)
	$(WITH_SDACCEL) $(CXX) $(HOST_CFLAGS) -MM -MP -MT $@ -MF $(@:.o=.d) $<
	$(WITH_SDACCEL) $(CXX) $(HOST_CFLAGS) -c $< -o $@

-include $(OBJ)/$(HOST_SRCS:%.cpp=%.d)

$(BIT)/$(CSIM_XCLBIN): $(SRC)/$(KERNEL_SRCS) $(BIN)/emconfig.json
	@mkdir -p $(BIT)
	$(WITH_SDACCEL) $(CLCXX) $(CLCXX_CSIM_OPT) $(CLCXX_OPT) -o $@ $<
	@rm -rf $$(ls -d .Xil/* 2>/dev/null|grep -vE "\-($$(pgrep xocc|tr '\n' '|'))-")
	@rmdir .Xil --ignore-fail-on-non-empty 2>/dev/null; exit 0

$(BIT)/$(COSIM_XCLBIN): $(SRC)/$(KERNEL_SRCS) $(BIN)/emconfig.json
	@mkdir -p $(BIT)
	@mkdir -p $(RPT)
	$(WITH_SDACCEL) $(CLCXX) $(CLCXX_COSIM_OPT) $(CLCXX_OPT) -o $@ $<
	@rm -rf $$(ls -d .Xil/* 2>/dev/null|grep -vE "\-($$(pgrep xocc|tr '\n' '|'))-")
	@rmdir .Xil --ignore-fail-on-non-empty 2>/dev/null; exit 0

$(BIT)/$(HW_XCLBIN): $(OBJ)/$(HW_XCLBIN:.xclbin=.xo)
	@mkdir -p $(BIT)
	$(WITH_SDACCEL) $(CLCXX) $(CLCXX_HW_OPT) $(CLCXX_OPT) -l -o $@ $<
	@rm -rf $$(ls -d .Xil/* 2>/dev/null|grep -vE "\-($$(pgrep xocc|tr '\n' '|'))-")
	@rmdir .Xil --ignore-fail-on-non-empty 2>/dev/null; exit 0

$(OBJ)/$(HW_XCLBIN:.xclbin=.xo): $(SRC)/$(KERNEL_SRCS)
	@mkdir -p $(OBJ)
	$(WITH_SDACCEL) $(CLCXX) $(CLCXX_HW_OPT) $(CLCXX_OPT) -c -o $@ $<
	@mkdir -p $(RPT)/$(HW_XCLBIN:.xclbin=)
	@cp _xocc_compile_$(KERNEL_SRCS:%.cpp=%)_$(HW_XCLBIN:%.xclbin=%.dir)/impl/kernels/$(KERNEL_NAME)/$(KERNEL_NAME)/solution_OCL_REGION_0/syn/report/*.rpt $(RPT)/$(HW_XCLBIN:.xclbin=)
	@cp _xocc_compile_$(KERNEL_SRCS:%.cpp=%)_$(HW_XCLBIN:%.xclbin=%.dir)/impl/kernels/$(KERNEL_NAME)/$(KERNEL_NAME)/solution_OCL_REGION_0/syn/report/*.rpt $(RPT)/$(HW_XCLBIN:.xclbin=)
	@rm -rf $$(ls -d .Xil/* 2>/dev/null|grep -vE "\-($$(pgrep xocc|tr '\n' '|'))-")
	@rmdir .Xil --ignore-fail-on-non-empty 2>/dev/null; exit 0

$(BIN)/emconfig.json:
	@mkdir -p $(BIN)
	cd $(BIN);$(WITH_SDACCEL) emconfigutil --platform $(XDEVICE) $(DEVICE_REPO_OPT) --od .

