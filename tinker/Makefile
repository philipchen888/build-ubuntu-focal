export BOARD=tinker
export LB_BOOTSTRAP_INCLUDE="apt-transport-https gnupg"
export OUT=$(PWD)/out
export BOOT=$(OUT)/boot.img
export ARCH=arm
export CROSS_COMPILE=/usr/local/gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf-

all: u-boot kernel rootfs mksdcardimg

cleanall:
	cd debian_u-boot; \
	make ARCH=arm clean
	cd linux-5.10; \
	sudo make mrproper
	sudo lb clean --purge
	\rm -rf config
	\rm -rf config linaro-rootfs.img

rootfs: clean prebuild buildrootfs postbuild mkimage

clean:
	sudo lb clean --purge
	\rm -rf config

prebuild:
	lb config \
	--apt-indices false \
	--apt-recommends false \
	--apt-secure false \
	--architectures armhf \
	--archive-areas 'main restricted universe multiverse' \
	--backports false \
	--binary-filesystem ext4 \
	--binary-images tar \
	--bootappend-live "hostname=linaro-alip username=linaro" \
	--bootstrap-qemu-arch armhf \
	--bootstrap-qemu-static /usr/bin/qemu-arm-static \
	--cache false \
	--chroot-filesystem none \
	--compression gzip \
	--debootstrap-options "--variant=minbase --include=apt-transport-https,gnupg" \
	--distribution focal \
	--gzip-options '-9 --rsyncable' \
	--iso-publisher 'Linaro; http://www.linaro.org/; linaro-dev@lists.linaro.org' \
	--iso-volume 'Linaro Focal $(date +%Y%m%d-%H:%M)' \
	--linux-flavours none \
	--linux-packages none \
	--parent-mirror-binary 'http://ports.ubuntu.com/ubuntu-ports/' \
	--parent-mirror-binary-security 'http://ports.ubuntu.com/ubuntu-ports/' \
	--parent-mirror-bootstrap 'http://ports.ubuntu.com/ubuntu-ports/' \
	--parent-mirror-chroot 'http://ports.ubuntu.com/ubuntu-ports/' \
	--parent-mirror-chroot-security 'http://ports.ubuntu.com/ubuntu-ports/' \
	--mode ubuntu \
	--security true \
	--system normal \
	--updates true

buildrootfs:
	\cp -rf $(PWD)/../customization/* config/
	sudo lb build

postbuild:
	ARCH=armhf $(PWD)/../mk-rootfs-ubuntu.sh

mkimage:
	$(PWD)/../mk-image.sh

u-boot: builduboot postbuilduboot
kernel: buildkernel postbuildkernel

builduboot:
	cd debian_u-boot; \
	make ARCH=arm clean; \
	make ARCH=arm tinker-rk3288_defconfig; \
	make ARCH=arm all

postbuilduboot:
	cd debian_u-boot; \
	./tools/mkimage -n rk3288 -T rksd -d ./spl/u-boot-spl-dtb.bin u-boot.img; \
	cat ./u-boot.bin >> u-boot.img

buildkernel:
	cd linux-5.10; \
	sudo make mrproper; \
	make multi_v7_defconfig; \
	make all -j4; \
	sudo make ARCH=arm INSTALL_MOD_PATH=./tmp modules_install

postbuildkernel:
	cd linux-5.10; \
	\rm -f $(BOOT); \
	mkfs.vfat -n "boot" -S 512 -C ${BOOT} 102400; \
	mmd -i ${BOOT} ::/extlinux; \
	mcopy -i ${BOOT} -s ../rk3288.conf ::/extlinux/extlinux.conf; \
	mcopy -i ${BOOT} -s ../hw_intf.conf ::; \
	mcopy -i ${BOOT} -s ./arch/arm/boot/zImage ::; \
	mcopy -i ${BOOT} -s ./arch/arm/boot/dts/rk3288-tinker.dtb ::

mksdcardimg:
	./mksdcardimg.sh
