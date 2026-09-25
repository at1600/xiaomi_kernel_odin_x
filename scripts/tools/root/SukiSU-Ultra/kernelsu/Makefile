kernelsu-objs := ksu.o
kernelsu-objs += allowlist.o
kernelsu-objs += dynamic_manager.o
kernelsu-objs += app_profile.o
kernelsu-objs += apk_sign.o
kernelsu-objs += sucompat.o
kernelsu-objs += syscall_hook_manager.o
kernelsu-objs += throne_tracker.o
kernelsu-objs += pkg_observer.o
kernelsu-objs += setuid_hook.o
kernelsu-objs += lsm_hooks.o
kernelsu-objs += kernel_compat.o
kernelsu-objs += kernel_umount.o
kernelsu-objs += supercalls.o
kernelsu-objs += feature.o
kernelsu-objs += throne_tracker.o
kernelsu-objs += ksud.o
kernelsu-objs += embed_ksud.o
kernelsu-objs += seccomp_cache.o
kernelsu-objs += file_wrapper.o
kernelsu-objs += throne_comm.o

kernelsu-objs += selinux/selinux.o
kernelsu-objs += selinux/sepolicy.o
kernelsu-objs += selinux/rules.o

ccflags-y += -I$(srctree)/security/selinux -I$(srctree)/security/selinux/include
ccflags-y += -I$(objtree)/security/selinux -include $(srctree)/include/uapi/asm-generic/errno.h

obj-$(CONFIG_KSU) += kernelsu.o
obj-$(CONFIG_KSU_TRACEPOINT_HOOK) += ksu_trace_export.o

obj-$(CONFIG_KPM) += kpm/

REPO_OWNER := SukiSU-Ultra
REPO_NAME := SukiSU-Ultra
REPO_BRANCH := main
KSU_VERSION_API := 4.1.1

KSU_VERSION := 40496
KSU_VERSION_FULL := $(KSU_VERSION)

ccflags-y += -DKSU_VERSION=$(KSU_VERSION)
ccflags-y += -DKSU_VERSION_FULL=\"$(KSU_VERSION_FULL)\"

# RKSU: checks for available hook
## Logic flipped for HAVE_KSU_HOOK: 0 is success, 1 is failure, but not with KSU_DRY_RUN
HAVE_KSU_HOOK ?= 1
KSU_DRY_RUN ?= 0

# Checks hooks state
ifeq ($(CONFIG_KSU_SHOULD_USE_NEW_TP), y)
$(info -- KernelSU: SHOULD_USE_NEW_TP)
ccflags-y += -DKSU_SHOULD_USE_NEW_TP
# Let's make it 0, so it would pass.
HAVE_KSU_HOOK := 0
endif

ifeq ($(CONFIG_KSU_MANUAL_HOOK), y)
HAVE_KSU_HOOK := $(shell grep -q "ksu_handle_faccessat" $(srctree)/fs/open.c; echo $$?)
ifeq ($(HAVE_KSU_HOOK),0)
$(info -- KernelSU: CONFIG_KSU_MANUAL_HOOK)
endif
endif

ifeq ($(KSU_DRY_RUN),0)
ifneq ($(HAVE_KSU_HOOK),0)
$(error -- KernelSU: No hooks were defined, bail!)
endif
else
$(info -- KernelSU in dry run mode, skip hook checks)

endif

# SELinux drivers check
ifeq ($(shell grep -q "current_sid(void)" $(srctree)/security/selinux/include/objsec.h; echo $$?),0)
ccflags-y += -DKSU_COMPAT_HAS_CURRENT_SID
endif
ifeq ($(shell grep -q "struct selinux_state " $(srctree)/security/selinux/include/security.h; echo $$?),0)
ccflags-y += -DKSU_COMPAT_HAS_SELINUX_STATE
endif

# Handle optional backports
ifeq ($(shell grep -q "strncpy_from_user_nofault" $(srctree)/include/linux/uaccess.h; echo $$?),0)
ccflags-y += -DKSU_OPTIONAL_STRNCPY
endif
ifeq ($(shell grep -q "ssize_t kernel_read" $(srctree)/fs/read_write.c; echo $$?),0)
ccflags-y += -DKSU_OPTIONAL_KERNEL_READ
endif
ifeq ($(shell grep "ssize_t kernel_write" $(srctree)/fs/read_write.c | grep -q "const void" ; echo $$?),0)
ccflags-y += -DKSU_OPTIONAL_KERNEL_WRITE
endif
ifeq ($(shell grep -q "int\s\+path_umount" $(srctree)/fs/namespace.c; echo $$?),0)
ccflags-y += -DKSU_HAS_PATH_UMOUNT
endif
ifeq ($(shell grep -q "inode_security_struct\s\+\*selinux_inode" $(srctree)/security/selinux/include/objsec.h; echo $$?),0)
ccflags-y += -DKSU_OPTIONAL_SELINUX_INODE
endif
ifeq ($(shell grep -q "task_security_struct\s\+\*selinux_cred" $(srctree)/security/selinux/include/objsec.h; echo $$?),0)
ccflags-y += -DKSU_OPTIONAL_SELINUX_CRED
endif
# seccomp_types.h were added on 6.7
ifeq ($(shell grep -q "atomic_t\s\+filter_count" $(srctree)/include/linux/seccomp.h $(srctree)/include/linux/seccomp_types.h; echo $$?),0)
ccflags-y += -DKSU_OPTIONAL_SECCOMP_FILTER_CNT
endif
# some old kernel backport this, let's check if put_seccomp_filter still exist
ifneq ($(shell grep -wq "put_seccomp_filter" $(srctree)/kernel/seccomp.c $(srctree)/include/linux/seccomp.h; echo $$?),0)
ccflags-y += -DKSU_OPTIONAL_SECCOMP_FILTER_RELEASE
endif
ifeq ($(shell grep -q "anon_inode_getfd_secure" $(srctree)/fs/anon_inodes.c; echo $$?),0)
ccflags-y += -DKSU_HAS_GETFD_SECURE
endif
ifeq ($(shell grep -A1 "^int vfs_getattr" $(srctree)/fs/stat.c | grep -q "query_flags"; echo $$?),0)
ccflags-y += -DKSU_HAS_NEW_VFS_GETATTR
endif

# Checks Samsung
ifeq ($(shell grep -q "CONFIG_KDP_CRED" $(srctree)/kernel/cred.c; echo $$?),0)
ccflags-y += -DSAMSUNG_UH_DRIVER_EXIST
endif
ifeq ($(shell grep -q "SEC_SELINUX_PORTING_COMMON" $(srctree)/security/selinux/avc.c; echo $$?),0)
ccflags-y += -DSAMSUNG_SELINUX_PORTING
endif
# Function proc_ops check
ifeq ($(shell grep -q "struct proc_ops " $(srctree)/include/linux/proc_fs.h; echo $$?),0)
ccflags-y += -DKSU_COMPAT_HAS_PROC_OPS
endif

# Custom Signs
ifdef KSU_EXPECTED_SIZE
ccflags-y += -DEXPECTED_SIZE=$(KSU_EXPECTED_SIZE)
$(info -- Custom KernelSU Manager signature size: $(KSU_EXPECTED_SIZE))
endif
ifdef KSU_EXPECTED_HASH
ccflags-y += -DEXPECTED_HASH=\"$(KSU_EXPECTED_HASH)\"
$(info -- Custom KernelSU Manager signature hash: $(KSU_EXPECTED_HASH))
endif

ifdef KSU_MANAGER_PACKAGE
ccflags-y += -DKSU_MANAGER_PACKAGE=\"$(KSU_MANAGER_PACKAGE)\"
$(info -- SukiSU Manager package name: $(KSU_MANAGER_PACKAGE))
endif

$(info -- Supported Unofficial Manager: 5ec1cff (GKI) ShirkNeko udochina (GKI and KPM))
# Kernel version check
KERNEL_VERSION := $(VERSION).$(PATCHLEVEL)
KERNEL_TYPE := Non-GKI
# Check for GKI 2.0 (5.10+ or 6.x+)
ifneq ($(shell test \( $(VERSION) -ge 5 -a $(PATCHLEVEL) -ge 10 \) -o $(VERSION) -ge 6; echo $$?),0)
# Check for GKI 1.0 (5.4)
ifeq ($(shell test $(VERSION)-$(PATCHLEVEL) = 5-4; echo $$?),0)
KERNEL_TYPE := GKI 1.0
endif
else
KERNEL_TYPE := GKI 2.0
endif
$(info -- KERNEL_VERSION: $(KERNEL_VERSION))
$(info -- KERNEL_TYPE: $(KERNEL_TYPE))

$(info -- KERNEL_VERSION: $(KERNEL_VERSION))
ifeq ($(CONFIG_KPM), y)
$(info -- KPM is enabled)
else
$(info -- KPM is disabled)
endif

ccflags-y += -Wno-strict-prototypes -Wno-int-conversion -Wno-gcc-compat
ccflags-y += -Wno-declaration-after-statement -Wno-unused-function -Wno-missing-prototypes

# Keep a new line here!! Because someone may append config
