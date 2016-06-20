
ifndef CONFIG_F2FS_FS
  EXTERNAL_BUILD=y
  CONFIG_F2FS_FS=m
  # Uncomment below to do debug build.
  #CONFIG_NILFS_DEBUG=y
endif

obj-$(CONFIG_F2FS_FS) += f2fs.o

f2fs-y		:= dir.o file.o inode.o namei.o hash.o super.o inline.o
f2fs-y		+= checkpoint.o gc.o data.o node.o segment.o recovery.o
f2fs-$(CONFIG_F2FS_STAT_FS) += debug.o
f2fs-$(CONFIG_F2FS_FS_XATTR) += xattr.o
f2fs-$(CONFIG_F2FS_FS_POSIX_ACL) += acl.o

list-$(CONFIG_F2FS_FS) += f2fs

# Some assembler options.
ifneq ($(CONFIG_F2FS_DEBUG),)
  EXTRA_CFLAGS += -g -fno-omit-frame-pointer
endif

EXTRA_CFLAGS += -I ../../include

#
# Only support external build, and does not support kernel build.
#
ifneq ($(KERNELRELEASE),)

ifeq ($(EXTERNAL_BUILD),y)
  EXTRA_CFLAGS += -U CONFIG_NILFS_XATTR       # not supported yet
  EXTRA_CFLAGS += -U CONFIG_NILFS_POSIX_ACL   # not supported yet

  ifneq ($(CONFIG_NILFS_DEBUG),)
    EXTRA_CFLAGS += -D CONFIG_NILFS_DEBUG
  endif
endif

else  # KERNELRELEASE

#
# Description for external build begins here
#

KVER := $(shell uname -r)
KSRC := /lib/modules/$(KVER)/build
MDIR := /lib/modules/$(KVER)/kernel/fs/f2fs/
PWD := $(shell pwd)

PREINC := -I$(PWD)/../../include
CFLAGS += -Wall

default:
	$(MAKE) -C $(KSRC) SUBDIRS=$(PWD) BUILD_DIR=$(PWD) \
	NOSTDINC_FLAGS=$(PREINC) modules

install: unload
	install -d $(MDIR)
	install -m 644 -c $(addsuffix .ko,$(list-m)) $(MDIR)
	/sbin/depmod $(KVER) -A

uninstall: unload
	rm -rf $(addprefix $(MDIR),$(addsuffix .ko,$(list-m)))
	/sbin/depmod $(KVER) -A

load:
	modprobe $(list-m)

unload:
	@test "`uname -r`" != "$(KVER)" || \
	     if [ -n "`mount -t $(list-m)`" ]; then \
	         echo "Unmount $(list-m) first."; false; \
	     elif (/sbin/lsmod | grep -e ^$(list-m)' ') > /dev/null; then /sbin/rmmod $(list-m); \
	     	 if [ $$? != 0 ]; then echo "failed to unload $(list-m)"; false; \
	         else echo "unloaded $(list-m)"; fi \
	     fi \

clean:
	rm -f *.mod.c *.mod *.o *.ko .*.cmd .*.flags *.lst *~ .#*
	rm -f modules.order Module.symvers
	rm -rf .tmp_versions .$(obj-m:.o=)*

endif # End of KERNELRELEASE
