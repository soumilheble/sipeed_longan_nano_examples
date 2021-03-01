#####################################################
# Sipeed Longan Nano - GD32VF103CBT6 Board makefile
# 
# Author: 
# 	Soumil Krishnanand Heble
# Website:
# 	decryptronics.github.io
#####################################################

########## Directory Tree ##########
# Project_Dir/
# -- Makefile (this file)
# -- include/
#    -- systick.h
#    -- gd32vf103_libopt.h
# -- src/
#    -- main.c
#    -- systick.c
# -- build/
#    -- obj/
#    -- lib/
#       -- libRISCV.a
#       -- libstd_peripheral.a
#       -- libusbfs_driver.a
#    -- target/

########## RISC-V TOOLS ##########
AR := riscv32-unknown-elf-ar
AS := riscv32-unknown-elf-as
CC := riscv32-unknown-elf-gcc
CXX := riscv32-unknown-elf-g++
CPP := riscv32-unknown-elf-cpp
LD := riscv32-unknown-elf-ld
OCPY := riscv32-unknown-elf-objcopy
RLIB := riscv32-unknown-elf-ranlib
DFU_UTIL := dfu-util

# Path to Firmware Library from GigaDevice
# Source: https://www.gigadevice.com/microcontroller/gd32vf103cbt6/
GD_FWDIR := /home/soumilheble/Projects/gd32vf103_firmware_lib_v1_0_2

# Name of ELF/Binary/Hex
PROJ_NAME := ledblinky

########## Directories ##########
MKDIR_P := mkdir -p
RM := rm -rf
### Include Directories
# Firmware Include Directories
fwincdir := Firmware/GD32VF103_standard_peripheral
fwincdir += Firmware/GD32VF103_standard_peripheral/Include
#fwincdir += Firmware/GD32VF103_usbfs_driver/Include
fwincdir += Firmware/RISCV/drivers
fwincdir += Firmware/RISCV/stubs
fwincdir := $(addprefix -I$(GD_FWDIR)/,$(fwincdir))
# User Include Directories
usrincdir := $(CURDIR)/include
usrincdir := $(addprefix -I,$(usrincdir))

### Source Directories
# RISCV
srvdir := Firmware/RISCV/drivers
srvdir += Firmware/RISCV/env_Eclipse
srvdir += Firmware/RISCV/stubs
srvdir := $(addprefix $(GD_FWDIR)/,$(srvdir))
# Standard Peripheral
sstdperiphdir := Firmware/GD32VF103_standard_peripheral
sstdperiphdir += Firmware/GD32VF103_standard_peripheral/Source
sstdperiphdir := $(addprefix $(GD_FWDIR)/,$(sstdperiphdir))
# USB File System
#susbfsdir := Firmware/GD32VF103_usbfs_driver/Source
#susbfsdir := $(addprefix $(GD_FWDIR)/,$(susbfsdir))
# User Source Directories
susrdir := $(CURDIR)/src

### Result Directories
# Object Directory
objdir := $(CURDIR)/build/obj
# Library Directory
libdir := $(CURDIR)/build/lib
# Target Dirctory
tgtdir := $(CURDIR)/build/target

########## CPPFLAGS - Preprocessor Flags ###########
CPPFLAGS := -DGD32VF103V_EVAL
CPPFLAGS += $(fwincdir) $(usrincdir)

########## CFLAGS - Compiler Flags ###########
CFLAGS := -c -std=gnu11 -Wall -Os
CFLAGS += -fmessage-length=0 -fsigned-char -ffunction-sections -fdata-sections -fno-common

### Uncomment for Debug Information
#CFLAGS += -g 
#CFLAGS += -v --save-temps

########## TARGET_ARCH - Target Spec ##########
TARGET_ARCH := -march=rv32imac -mabi=ilp32 -mcmodel=medlow

########## ARFLAGS - Archive ##########
ARFLAGS := rc

########## LDFLAGS - Linker Flags ##########
LDFLAGS := -T $(GD_FWDIR)/Firmware/RISCV/env_Eclipse/GD32VF103xB.lds
LDFLAGS += -nostartfiles -Xlinker --gc-sections --specs=nano.specs
LDFLAGS += -L$(libdir) -Wl,--start-group -lc -lc -Wl,--end-group

########## PATTERN RULES ##########
$(objdir)/%.o: %.c
	$(MKDIR_P) $(@D)
	$(COMPILE.c) -o $@ $<

$(objdir)/%.o: %.S
	$(MKDIR_P) $(@D)
	$(COMPILE.S) -o $@ $<

%.a:
	$(MKDIR_P) $(libdir)
	$(AR) $(ARFLAGS) $@ $(patsubst $(objdir)/,,$^)
	$(RLIB) $@

%.elf:
	$(MKDIR_P) $(@D)
	$(LINK.o) $^ $(LOADLIBES) $(LDLIBS) -o $@

%.bin: %.elf
	$(MKDIR_P) $(@D)
	$(OCPY) -O binary $< $@

%.hex: %.elf
	$(MKDIR_P) $(@D)
	$(OCPY) -O ihex $< $@

########## EXPLICIT RULES ##########

.PHONY: all clean cleanall upload

all: $(tgtdir)/$(PROJ_NAME).hex $(tgtdir)/$(PROJ_NAME).bin

clean:
	$(RM) $(objdir)
	$(RM) $(libdir)

cleanall:
	$(RM) $(CURDIR)/build

upload: $(tgtdir)/$(PROJ_NAME).bin
	dfu-util -d 28e9:0189 -a 0 --dfuse-address 0x08000000:leave -D $(tgtdir)/$(PROJ_NAME).bin

$(tgtdir)/$(PROJ_NAME).hex: $(tgtdir)/$(PROJ_NAME).elf
$(tgtdir)/$(PROJ_NAME).bin: $(tgtdir)/$(PROJ_NAME).elf
#$(tgtdir)/$(PROJ_NAME).elf: $(libdir)/libRISCV.a $(libdir)/libstd_peripheral.a $(libdir)/libusbfs_driver.a
$(tgtdir)/$(PROJ_NAME).elf: $(addprefix $(objdir)/,$(patsubst %.c,%.o,$(wildcard $(addsuffix /*.c,$(susrdir)))))
$(tgtdir)/$(PROJ_NAME).elf: $(libdir)/libRISCV.a $(libdir)/libstd_peripheral.a
$(libdir)/libRISCV.a: $(addprefix $(objdir)/,$(patsubst %.c,%.o,$(wildcard $(addsuffix /*.c,$(srvdir)))))
$(libdir)/libRISCV.a: $(addprefix $(objdir)/,$(patsubst %.S,%.o,$(wildcard $(addsuffix /*.S,$(srvdir)))))
$(libdir)/libstd_peripheral.a: $(addprefix $(objdir)/,$(patsubst %.c,%.o,$(wildcard $(addsuffix /*.c,$(sstdperiphdir)))))
#$(libdir)/libusbfs_driver.a: $(addprefix $(objdir)/,$(patsubst %.c,%.o,$(wildcard $(addsuffix /*.c,$(susbfsdir)))))

