# This file is included by the global makefile so that you can add your own
# architecture-specific flags and dependencies. Remember to do have actions
# for "archclean" and "archdep" for cleaning up and making dependencies for
# this architecture
#
# This file is subject to the terms and conditions of the GNU General Public
# License.  See the file "COPYING" in the main directory of this archive
# for more details.
#

OBJCOPYFLAGS    := -O binary
LDFLAGS_vmlinux :=
ifeq ($(CONFIG_DYNAMIC_FTRACE),y)
	LDFLAGS_vmlinux := --no-relax
	KBUILD_CPPFLAGS += -DCC_USING_PATCHABLE_FUNCTION_ENTRY
	CC_FLAGS_FTRACE := -fpatchable-function-entry=8
endif

ifeq ($(CONFIG_64BIT)$(CONFIG_CMODEL_MEDLOW),yy)
KBUILD_CFLAGS_MODULE += -mcmodel=medany
endif

export BITS
ifeq ($(CONFIG_ARCH_RV64I),y)
	BITS := 64
	UTS_MACHINE := riscv64

	KBUILD_CFLAGS += -mabi=lp64
	KBUILD_AFLAGS += -mabi=lp64

	KBUILD_LDFLAGS += -melf64lriscv
else
	BITS := 32
	UTS_MACHINE := riscv32

	KBUILD_CFLAGS += -mabi=ilp32
	KBUILD_AFLAGS += -mabi=ilp32
	KBUILD_LDFLAGS += -melf32lriscv
endif

# ISA string setting
riscv-march-cflags-$(CONFIG_ARCH_RV32I)		:= rv32ima
riscv-march-cflags-$(CONFIG_ARCH_RV64I)		:= rv64ima
riscv-march-cflags-$(CONFIG_RISCV_ISA_C)	:= $(riscv-march-cflags-y)c

riscv-march-aflags-$(CONFIG_ARCH_RV32I)		:= rv32ima
riscv-march-aflags-$(CONFIG_ARCH_RV64I)		:= rv64ima
riscv-march-aflags-$(CONFIG_FPU)		:= $(riscv-march-aflags-y)fd
riscv-march-aflags-$(CONFIG_RISCV_ISA_C)	:= $(riscv-march-aflags-y)c
riscv-march-aflags-$(CONFIG_VECTOR_1_0)		:= $(riscv-march-aflags-y)v
riscv-march-aflags-$(CONFIG_VECTOR_0_7)		:= $(riscv-march-aflags-y)v0p7
riscv-march-aflags-$(CONFIG_RISCV_ISA_THEAD)	:= $(riscv-march-aflags-y)xthead

KBUILD_CFLAGS += -march=$(riscv-march-cflags-y) -Wa,-march=$(riscv-march-aflags-y)
KBUILD_AFLAGS += -march=$(riscv-march-aflags-y)

KBUILD_CFLAGS += -mno-save-restore
KBUILD_CFLAGS += -DCONFIG_PAGE_OFFSET=$(CONFIG_PAGE_OFFSET)

machine-$(CONFIG_ARCH_CVITEK)       += cvitek

ifeq ($(CONFIG_CMODEL_MEDLOW),y)
	KBUILD_CFLAGS += -mcmodel=medlow
endif
ifeq ($(CONFIG_CMODEL_MEDANY),y)
	KBUILD_CFLAGS += -mcmodel=medany
endif
ifeq ($(CONFIG_PERF_EVENTS),y)
        KBUILD_CFLAGS += -fno-omit-frame-pointer
endif

KBUILD_CFLAGS_MODULE += $(call cc-option,-mno-relax)

# GCC versions that support the "-mstrict-align" option default to allowing
# unaligned accesses.  While unaligned accesses are explicitly allowed in the
# RISC-V ISA, they're emulated by machine mode traps on all extant
# architectures.  It's faster to have GCC emit only aligned accesses.
KBUILD_CFLAGS += $(call cc-option,-mstrict-align)

# arch specific predefines for sparse
CHECKFLAGS += -D__riscv -D__riscv_xlen=$(BITS)

# Default target when executing plain make
boot		:= arch/riscv/boot
KBUILD_IMAGE	:= $(boot)/Image.gz

head-y := arch/riscv/kernel/head.o

core-y += arch/riscv/

libs-y += arch/riscv/lib/
libs-$(CONFIG_EFI_STUB) += $(objtree)/drivers/firmware/efi/libstub/lib.a

PHONY += vdso_install
vdso_install:
	$(Q)$(MAKE) $(build)=arch/riscv/kernel/vdso $@
	$(if $(CONFIG_COMPAT),$(Q)$(MAKE) \
		$(build)=arch/riscv/kernel/compat_vdso $@)

ifeq ($(CONFIG_MMU),y)
prepare: vdso_prepare
vdso_prepare: prepare0
	$(Q)$(MAKE) $(build)=arch/riscv/kernel/vdso include/generated/vdso-offsets.h
	$(if $(CONFIG_COMPAT),$(Q)$(MAKE) \
		$(build)=arch/riscv/kernel/compat_vdso include/generated/compat_vdso-offsets.h)
endif

ifeq ($(CONFIG_RISCV_M_MODE)$(CONFIG_SOC_KENDRYTE),yy)
KBUILD_IMAGE := $(boot)/loader.bin
else
KBUILD_IMAGE := $(boot)/Image.gz
endif
BOOT_TARGETS := Image Image.gz loader loader.bin

all:	$(notdir $(KBUILD_IMAGE))

$(BOOT_TARGETS): vmlinux
	$(Q)$(MAKE) $(build)=$(boot) $(boot)/$@
	@$(kecho) '  Kernel: $(boot)/$@ is ready'

zinstall install:
	$(Q)$(MAKE) $(build)=$(boot) $@
