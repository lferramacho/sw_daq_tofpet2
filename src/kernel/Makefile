ifneq ("$(wildcard /etc/redhat-release)","")
	EXTRA_CFLAGS += -DPETSYS_RHEL_PATCH
endif

obj-m := psdaq.o

path := $(shell uname -r)

all:
	make -C /lib/modules/$(path)/build M=$(shell pwd)  modules
clean:
	make -C /lib/modules/$(path)/build M=$(shell pwd) clean
