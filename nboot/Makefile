#
# (C) Copyright 2000-2011
# Wolfgang Denk, DENX Software Engineering, wd@denx.de.
#
# (C) Copyright 2011
# Daniel Schwierzeck, daniel.schwierzeck@googlemail.com.
#
# (C) Copyright 2011
# Texas Instruments Incorporated - http://www.ti.com/
# Aneesh V <aneesh@ti.com>
#
# See the file COPYING in the root directory of the source tree for details.
#
# Based on top-level Makefile.
#

SKIP_AUTO_CONF:=yes
include $(TOPDIR)/mk/config.mk
include $(TOPDIR)/.module.common.mk
include $(TOPDIR)/board/$(CP_BOARD)/$(MAKECMDGOALS).mk

#This for Tina system, to copy bin to tina SDK
ifeq ($(TARGET_BUILD_VARIANT),tina)
tina_path=$(TOPDIR)/../../../target/allwinner/$(TARGET_PLATFORM)-common/bin
exist = $(shell if [ -d $(tina_path) ]; then echo "exist"; else echo "notexist"; fi;)
ifeq ($(exist),exist)
TARGETDIR=$(TOPDIR)/../../../target/allwinner/$(TARGET_PLATFORM)-common/bin
endif

tina_path=$(TOPDIR)/../../../device/config/chips/$(TARGET_PLATFORM)/bin
exist = $(shell if [ -d $(tina_path) ]; then echo "exist"; else echo "notexist"; fi;)
ifeq ($(exist),exist)
TARGETDIR=$(TOPDIR)/../../../device/config/chips/$(TARGET_PLATFORM)/bin
endif

else
#TOOLS_DIR  := $(TOPDIR)/../../../device/config
#TARGETDIR=$(TOPDIR)/../../../device/config/chips/$(PLATFORM)/bin
ifneq ($(LICHEE_BUSSINESS), )
	TARGETDIR=$(LICHEE_CHIP_CONFIG_DIR)/$(LICHEE_BUSSINESS)/bin
else
	TARGETDIR=$(LICHEE_CHIP_CONFIG_DIR)/bin
endif
endif

CPU ?= armv7
CPUDIR = $(TOPDIR)/arch/$(ARCH)/cpu/$(CPU)
BOOT0_LDSCRIPT := $(CPUDIR)/boot0.lds

# We want the final binaries in this directory
NBOOT_DIR := $(TOPDIR)/nboot/
obj := $(TOPDIR)/nboot/


LIBS-y  += arch/$(ARCH)/cpu/$(CPU)/libarch.o
LIBS-y += nboot/main/libmain.o
LIBS := $(addprefix $(TOPDIR)/,$(sort $(LIBS-y)))

A-LIBS-$(CFG_SUNXI_SDMMC)   := $(TOPDIR)/board/$(CP_BOARD)/lib$(PLATFORM)_sdcard.a
ifeq (x$(findstring y,$(CFG_SUNXI_NAND)$(CFG_SUNXI_SPINAND)), xy)
A-LIBS-y  := $(TOPDIR)/board/$(CP_BOARD)/lib$(PLATFORM)_nand.a
endif
A-LIBS-$(CFG_SUNXI_SPINOR)  := $(TOPDIR)/board/$(CP_BOARD)/lib$(PLATFORM)_spinor.a

A-OBJS := $(A-LIBS-y:%.a=%.o)

ifdef CFG_WEAK_SYMBOL
COBJS-y += $(A-OBJS)
else
COBJS-y += $(A-LIBS-y)
endif

COBJS-y += $(TOPDIR)/nboot/main/boot0_main.o

OBJS = $(sort $(COBJS-y))

ifndef CONFIG_BOOT0_SIZE_LIMIT
	BOOT0SIZE = $(CFG_SYS_INIT_RAM_SIZE)
else
	BOOT0SIZE = $(CFG_BOOT0_SIZE_LIMIT)
endif

# Special flags for CPP when processing the linker script.
# Pass the version down so we can handle backwards compatibility
# on the fly.
LDPPFLAGS += \
	-DBOOT0ADDR=$(CFG_BOOT0_RUN_ADDR)	 \
	-DBOOT0SIZE=$(BOOT0SIZE)	 \
	-DCPUDIR=$(CPUDIR) \
	$(shell $(LD) --version | \
	  sed -ne 's/GNU ld version \([0-9][0-9]*\)\.\([0-9][0-9]*\).*/-DLD_MAJOR=\1 -DLD_MINOR=\2/p')


spinor:	 $(LIBS) $(LIBSPINOR) boot0.lds
	$(Q)echo bootaddr is $(CFG_BOOT0_RUN_ADDR)
ifdef CFG_WEAK_SYMBOL
	$(OBJCOPY) $(CFG_WEAK_SYMBOL) $(A-LIBS-y) $(A-OBJS)
endif
	$(LD) $(OBJS) $(LIBSPINOR) $(PLATFORM_LIBGCC) $(LDFLAGS) $(LDFLAGS_GC) -Tboot0.lds -o boot0_spinor.elf -Map boot0_spinor.map
	$(OBJCOPY) $(OBJCFLAGS) -O binary  boot0_spinor.elf boot0_spinor.bin
	@$(TOPDIR)/mk/gen_check_sum $(SRCTREE)/nboot/boot0_spinor.bin boot0_spinor_$(PLATFORM)$(DRAM_TYPE_NAME).bin > /dev/null
	@$(TOPDIR)/mk/encrypto_boot0 -f boot0_spinor_$(PLATFORM)$(DRAM_TYPE_NAME).bin  -c $(LICHEE_IC) >/dev/null
	@-cp -v boot0_spinor_$(PLATFORM)$(DRAM_TYPE_NAME).bin $(TARGETDIR)/
	@-cp -v boot0_spinor_$(PLATFORM)$(DRAM_TYPE_NAME).bin $(LICHEE_PLAT_OUT)/

mmc:	 $(LIBS) $(LIBMMC) boot0.lds
	$(Q)echo bootaddr is $(CFG_BOOT0_RUN_ADDR)
ifdef CFG_WEAK_SYMBOL
	$(OBJCOPY) $(CFG_WEAK_SYMBOL) $(A-LIBS-y) $(A-OBJS)
endif
	$(LD) $(OBJS) $(LIBMMC) $(PLATFORM_LIBGCC) $(LDFLAGS) $(LDFLAGS_GC) -Tboot0.lds -o boot0_sdcard.elf -Map boot0_sdcard.map
	$(OBJCOPY) $(OBJCFLAGS) -O binary  boot0_sdcard.elf boot0_sdcard.bin
	@$(TOPDIR)/mk/gen_check_sum boot0_sdcard.bin boot0_sdcard_$(PLATFORM)$(DRAM_TYPE_NAME).bin > /dev/null
	@$(TOPDIR)/mk/encrypto_boot0 -f boot0_sdcard_$(PLATFORM)$(DRAM_TYPE_NAME).bin  -c $(LICHEE_IC) >/dev/null
	@-cp -v boot0_sdcard_$(PLATFORM)$(DRAM_TYPE_NAME).bin $(TARGETDIR)/
	@-cp -v boot0_sdcard_$(PLATFORM)$(DRAM_TYPE_NAME).bin $(LICHEE_PLAT_OUT)/

nand:	 $(LIBS) $(LIBNAND) boot0.lds
	$(Q)echo bootaddr is $(CFG_BOOT0_RUN_ADDR)
ifdef CFG_WEAK_SYMBOL
	$(OBJCOPY) $(CFG_WEAK_SYMBOL) $(A-LIBS-y) $(A-OBJS)
endif
	$(LD) $(OBJS) $(LIBNAND) $(PLATFORM_LIBGCC) $(LDFLAGS) $(LDFLAGS_GC) -Tboot0.lds -o boot0_nand.elf -Map boot0_nand.map
	$(OBJCOPY) $(OBJCFLAGS) -O binary  boot0_nand.elf boot0_nand.bin
	@$(TOPDIR)/mk/gen_check_sum boot0_nand.bin boot0_nand_$(PLATFORM)$(DRAM_TYPE_NAME).bin > /dev/null
	@$(TOPDIR)/mk/encrypto_boot0 -f boot0_nand_$(PLATFORM)$(DRAM_TYPE_NAME).bin  -c $(LICHEE_IC) >/dev/null
	@-cp -v boot0_nand_$(PLATFORM)$(DRAM_TYPE_NAME).bin $(TARGETDIR)/
	@-cp -v boot0_nand_$(PLATFORM)$(DRAM_TYPE_NAME).bin $(LICHEE_PLAT_OUT)/

mmc_$(DRAM_TYPE):mmc
nand_$(DRAM_TYPE):nand


$(LIBS): depend
	$(MAKE) -C $(dir $(subst $(OBJTREE),,$@))

$(LIBNAND): depend
	$(MAKE) -C $(dir $(subst $(OBJTREE),,$@))

$(LIBMMC): depend
	$(MAKE) -C $(dir $(subst $(OBJTREE),,$@))

$(LIBSPINOR): depend
	$(MAKE) -C $(dir $(subst $(OBJTREE),,$@))

boot0.lds: $(BOOT0_LDSCRIPT)
	$(Q)$(CPP) $(ALL_CFLAGS) $(LDPPFLAGS) -ansi -D__ASSEMBLY__ -P - <$^ >$@

$(NBOOT_DIR)cur.log:
	@git show HEAD --pretty=format:"%H" | head -n 1 > cur.log

sinclude $(TOPDIR)/mk/target_for_conf.mk
depend: .depend build-confs
#########################################################################

# defines $(NBOOT_DIR).depend target
include $(TOPDIR)/mk/rules.mk

sinclude .depend

#########################################################################
