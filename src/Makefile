# NOTICE: only tested in redhat 8.0/ centos 7.4

KERNEL_SOURCE_VERSION ?= $(shell uname -r)
KERNEL_TREE ?= /lib/modules/$(KERNEL_SOURCE_VERSION)/build

COMMIT_REV ?= $(shell git describe  --always --abbrev=12)
EXTRA_CFLAGS += -I$(KERNEL_TREE)/drivers/md -I./ -DCOMMIT_REV="\"$(COMMIT_REV)\""
#EXTRA_CFLAGS += -I$(KERNEL_TREE)/drivers/md -I./
EXTRA_CFLAGS += -I$(KERNEL_TREE)/include/ -I$(KERNEL_TREE)/include/linux 

obj-m += flashcache.o
flashcache-objs := flashcache_conf.o flashcache_main.o flashcache_subr.o flashcache_ioctl.o flashcache_procfs.o flashcache_kcopy.o flashcache_reclaim.o

.PHONY: all
all: modules utils

.PHONY:    modules
modules:
	make -C $(KERNEL_TREE) M=$(PWD) modules V=0
	#make CC=clang CFLANGS=-O3 -C $(KERNEL_TREE) M=$(PWD) modules V=0

.PHONY:    utils
utils:
	make -C utils all

.PHONY: modules_install
modules_install: modules
	install -o root -g root -m 0755 -d $(DESTDIR)/lib/modules/$(KERNEL_SOURCE_VERSION)/extra/flashcache/
	install -o root -g root -m 0755 flashcache.ko $(DESTDIR)/lib/modules/$(KERNEL_SOURCE_VERSION)/extra/flashcache/
	depmod -a $(KERNEL_SOURCE_VERSION)

.PHONY: utils_install
utils_install: utils
	make -C utils install

.PHONY: ocf_install
ocf_install:
	make -C ocf install

.PHONY: install
install: modules_install utils_install ocf_install

.PHONY: clean
clean:
	make -C $(KERNEL_TREE) M=$(PWD) clean
	make -C utils clean

.PHONY: rhel5-setup
rhel5-setup: $(RHEL5_TREE)
	make -C $(RHEL5_TREE) oldconfig ; \
	make -C $(RHEL5_TREE) prepare modules_prepare
	ln -s -f $(RHEL5_SRC)/Module.symvers $(RHEL5_TREE)/Module.symvers

$(RHEL5_TREE):
	rpmbuild -bp --target=`uname -m` $(RHEL5_SPEC) 2>&1 | tee `dirname $(RHEL5_SPEC)`/prep.log
