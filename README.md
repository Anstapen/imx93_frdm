# iMX9 FRDM Board Configuration Repository

This repository contains all needed files to 

1. compile a custom U-boot, based on https://github.com/Anstapen/uboot-imx[GitHub - Anstapen/uboot-imx: i.MX U-Boot](https://github.com/Anstapen/uboot-imx)

2. compile a custom Linux Kernel based on [GitHub - Anstapen/linux-imx: i.MX Linux kernel](https://github.com/Anstapen/linux-imx)

3. Create a RootFS

## General Information

Currently, we use source code (Ubbot, TF-A, Linux) that is based on the NXP Linux Software Release 6.6.52-2.0.0.

## Building U-Boot, Linux Kernel and RootFS

First, we are going to build the needed components using Buildroot.

1. Download Buildroot.

2. copy the defconfig located in this repository under `buildroot_configs` to the buildroot folder `configs`.

3. run `make imx93evk_defconfig` to set all the config variables.

4. run `make`

5. Uboot, ARM Trusted Firmware (TF-A), Linux Kernel and the RootFS should build.

## Combining Uboot, TF-A and SPL into one binary

For a detailed description of the Boot Flow, see Chapter 3.1 of NXP AN14093.

1. Clone the imx-mkimage repository `git clone https://github.com/nxp-imx/imx-mkimage`

2. Checkout the commit `lf-6.6.52-2.2.0`

3. Download and extract the NXP firmware (you can find the exact name in the NXP Linux Release Notes):
   
   ```
   wget https://www.nxp.com/lgfiles/NMG/MAD/YOCTO/firmware-imx-8.26-d4c33ab.bin
   chmod +x firmware-imx-8.26-d4c33ab.bin
   ./firmware-imx-8.26-d4c33ab.bin
   ```

4. Download and extrace the Edgelock Secure Enclave firmware (ELE):
   
   ```
   wget https://www.nxp.com/lgfiles/NMG/MAD/YOCTO/firmware-ele-imx-1.3.0-17945fc.bin
   chmod +x firmware-ele-imx-1.3.0-17945fc.bin
   ./firmware-ele-imx-1.3.0-17945fc.bin
   ```

5. Add the needed files to the imx-mkimage folder:
   
   ```
   cd imx-mkimage/iMX93
   # from the buildroot output/images folder
   bl31.bin
   u-boot.bin
   u-boot-spl.bin
   imx93-11x11-evk.dtb
   # From the NXP firmware folder
   firmware-imx-X.XX/firmware/ddr/synopsys/*
   # From the NXP ELE firmware folder
   mx93a1-ahab-container.img
   # The mkimage app from
   buildroot/output/build/uboot-XX.XX/tools/mkimage
   ```

6. Invoke ``make SOC=iMX93 flash_singleboot`` to create the binary

7. Connect the FRDM Board via USB and execute `sudo ~/Downloads/uuu -b sd flash.bin` to copy the bootloader binary to the target.

## Setting up a Network File System

```
$ sudo apt-get install nfs-kernel-server
$ sudo mkdir /home/${USER}/nfs
$ sudo chmod 777 /home/${USER}/nfs
```

```
$ sudo vim /etc/exports
/home/<user>/nfs *(rw,sync,no_root_squash,no_subtree_check)
```

```
$ sudo service nfs-kernel-server restart
```

Currently, the relevant kernel arguments are in "mmcargs":

```
setenv bootargs '${jh_clk}' '${mcore_clk}' console='${console}' root=/dev/nfs ip=192.168.10.6 nfsroot=192.168.10.2:/home/anton/Repositories/pedal_rootfs,nfsvers=3,tcp
```

## Enable SSH on the target

OpenSSH needs to be enabled in Buildroot.

This needs to be enabled in `/etc/ssh/sshd_config` :

```
PermitRootLogin yes
```

Also,  password needs to be enabled for root.

## Developing custom Linux Kernel Drivers

When developing custom Linux Kernel Drivers be sure to execute the following commands in the buildroot directory:

```
make linux-rebuild
make
sudo tar -C ~/Repositories/pedal_rootfs/ -xf output/images/rootfs.tar
```

To use a local linux repository, create a file `local.mk` in the buildroot repository with the following content:

```
LINUX_OVERRIDE_SRCDIR=<path_to_linux_repo>
```
