#
# Kbuild for top-level directory of the kernel
# This file takes care of the following:
# 1) Generate bounds.h
# 2) Generate timeconst.h
# 3) Generate asm-offsets.h (may need bounds.h and timeconst.h)
# 4) Check for missing system calls

# Default sed regexp - multiline due to syntax constraints
define sed-y
	"/^->/{s:->#\(.*\):/* \1 */:; \
	s:^->\([^ ]*\) [\$$#]*\([-0-9]*\) \(.*\):#define \1 \2 /* \3 */:; \
	s:^->\([^ ]*\) [\$$#]*\([^ ]*\) \(.*\):#define \1 \2 /* \3 */:; \
	s:->::; p;}"
endef

# Use filechk to avoid rebuilds when a header changes, but the resulting file
# does not
define filechk_offsets
	(set -e; \
	 echo "#ifndef $2"; \
	 echo "#define $2"; \
	 echo "/*"; \
	 echo " * DO NOT MODIFY."; \
	 echo " *"; \
	 echo " * This file was generated by Kbuild"; \
	 echo " */"; \
	 echo ""; \
	 sed -ne $(sed-y); \
	 echo ""; \
	 echo "#endif" )
endef
BRANCH=android-4.4
CROSS_COMPILE=x86_64-linux-android-
DEFCONFIG=x86_64_ranchu_defconfig
EXTRA_CMDS=''
KERNEL_DIR=common
LINUX_GCC_CROSS_COMPILE_PREBUILTS_BIN=prebuilts/gcc/linux-x86/x86/x86_64-linux-android-4.9/bin
FILES="
arch/x86/boot/bzImage
vmlinux
System.map
#####
# 1) Generate bounds.h

bounds-file := include/generated/bounds.h

always  := $(bounds-file)
targets := kernel/bounds.s

# We use internal kbuild rules to avoid the "is up to date" message from make
kernel/bounds.s: kernel/bounds.c FORCE
	$(Q)mkdir -p $(dir $@)
	$(call if_changed_dep,cc_s_c)

$(obj)/$(bounds-file): kernel/bounds.s FORCE
	$(call filechk,offsets,__LINUX_BOUNDS_H__)

#####
# 2) Generate timeconst.h

timeconst-file := include/generated/timeconst.h

targets += $(timeconst-file)

quiet_cmd_gentimeconst = GEN     $@
define cmd_gentimeconst
	(echo $(CONFIG_HZ) | bc -q $< ) > $@
endef
define filechk_gentimeconst
	(echo $(CONFIG_HZ) | bc -q $< )
endef

$(obj)/$(timeconst-file): kernel/time/timeconst.bc FORCE
	$(call filechk,gentimeconst)

#####
# 3) Generate asm-offsets.h
#

offsets-file := include/generated/asm-offsets.h

always  += $(offsets-file)
targets += arch/$(SRCARCH)/kernel/asm-offsets.s

# We use internal kbuild rules to avoid the "is up to date" message from make
arch/$(SRCARCH)/kernel/asm-offsets.s: arch/$(SRCARCH)/kernel/asm-offsets.c \
                                      $(obj)/$(timeconst-file) $(obj)/$(bounds-file) FORCE
	$(Q)mkdir -p $(dir $@)
	$(call if_changed_dep,cc_s_c)

$(obj)/$(offsets-file): arch/$(SRCARCH)/kernel/asm-offsets.s FORCE
	$(call filechk,offsets,__ASM_OFFSETS_H__)

#####
# 4) Check for missing system calls
#

always += missing-syscalls
targets += missing-syscalls

quiet_cmd_syscalls = CALL    $<
      cmd_syscalls = $(CONFIG_SHELL) $< $(CC) $(c_flags) $(missing_syscalls_flags)

missing-syscalls: scripts/checksyscalls.sh $(offsets-file) FORCE
	$(call cmd,syscalls)

# Keep these three files during make clean
no-clean-files := $(bounds-file) $(offsets-file) $(timeconst-file)
