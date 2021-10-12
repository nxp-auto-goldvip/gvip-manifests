======================================
OpenEmbedded/Yocto project for GoldVIP
======================================

GoldVIP project manifest files are used together with GoldVIP Yocto meta layer to
build the NXP Gold Vehicle Integration Platform (GoldVIP).

First Time Setup
================

To get and build GoldVIP you need to have `repo` installed and its dependencies.
This only needs to be done once.

- Update the package manager (sudo apt update)

- Install `repo` tool dependencies:

   - Ubuntu 18.04 LTS or later
   - python 2.x - 2.6 or newer (``$ sudo apt-get install python``)
   - git 1.8.3 or newer (``$ sudo apt-get install git``)
   - curl (``$ sudo apt-get install curl``)

- Install `repo` tool::

   mkdir ~/bin
   curl http://commondatastorage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
   chmod a+x ~/bin/repo
   PATH=${PATH}:~/bin

Building GoldVIP
================

The following steps will build a GoldVIP image based on NXP Auto Linux BSP image.

Note:
A Yocto build needs at least 50GB of free space and takes a lot of time (a few 
hours, depending on the system configuration). It is recommended to use a 
powerful system with many cores and a fast storage media (e.g., SSD).
The recommended RAM size is 8 GB.

Download the Yocto project environment
--------------------------------------

Get the latest GoldVIP manifest files and bring other required repositories::

  mkdir nxp-yocto-goldvip
  cd nxp-yocto-goldvip
  repo init -u https://source.codeaurora.org/external/autobsps32/goldvip/gvip-manifests -b develop -m default.xml
  repo sync

Note: for a specific GoldVIP release or engineering build, please use the proper
branch (`-b <branch>`) and manifest file (`-m <manifest>`).

Note: to fetch internal development repositories use Bitbucket URL::

  repo init -u ssh://git@bitbucket.sw.nxp.com/gvip/gvip-manifests.git -b develop -m default-bitbucket.xml

Manifest files description:

 - default.xml -> fetch GoldVIP repositories from https://source.codeaurora.org
 - default-bitbucket.xml -> fetch internal development repositories from Bitbucket
 - alb.xml -> default NXP Auto Linux BSP without GoldVIP extensions

Setup the build environment
---------------------------

- Install all prerequisites before starting the Yocto build (first time only)::

   ./sources/meta-alb/scripts/host-prepare.sh
   sudo apt-get install libssl-dev

- Create a build directory and setup build environment::

   source nxp-setup-alb.sh -D fsl-goldvip -m <machine> -e "meta-aws meta-java"

  Currently, the only supported `<machine>` (NXP board) is: `s32g274ardb2`.

  PFE firmware, XEN hypervisor and bridging utilities are mandatory and
  configured by the fsl-goldvip distro.

  GoldVIP CAN Gateway, GoldVIP Bootloader binaries and SJA1110 firmware are
  also configured by the fsl-goldvip distro, but they are optional and can be
  removed from the image.

- Download GoldVIP binaries from your nxp.com account and append the following
  line to the file `build_<machine>/conf/local.conf`::

   GOLDVIP_BINARIES_DIR = "<path to the local GoldVIP binaries directory>"

- Download PFE firmware from your nxp.com account and append the following line
  to the file `build_<machine>/conf/local.conf`::

   NXP_FIRMWARE_LOCAL_DIR = "<path to the local s32g_pfe_class.fw file>"

Note: To remove GoldVIP CAN Gateway and GoldVIP Bootloader binaries,
append the following lines to the file `build_<machine>/conf/local.conf`::

   DISTRO_FEATURES_remove = "goldvip-can-gw"
   DISTRO_FEATURES_remove = "goldvip-bootloader"

Note: To remove SJA1110 firmware, append the following lines to
the file `build_<machine>/conf/local.conf`::

   SJA1110_UC_FW = ""
   SJA1110_SWITCH_FW = ""

Note: To use internal development GoldVIP repository add the following line in
`build_<machine>/conf/local.conf`::

  GOLDVIP_URL = "git://bitbucket.sw.nxp.com/gvip/gvip.git;protocol=ssh"

Build the image
---------------

::

  bitbake fsl-image-goldvip

Running the above command would be enough to completely build u-boot, kernel,
modules and a rootfs ready to be deployed. Look for a build result in
`build_<machine>/tmp/deploy/images/`.

Deploy the image
----------------

The file `<image-name>.sdcard` is a disk image with all necessary partitions and
contains the bootloader, kernel and rootfs. You can just low-level copy the data
on this file to the SD card device using dd as on the following command example::

  sudo dd if=fsl-image-goldvip-s32g274ardb2.sdcard of=/dev/<sd-device> bs=1M conv=fsync,notrunc status=progress && sync

Ensure that any partitions on the card are properly unmounted before writing
the card image, or you may have a corrupted card image in the end.
Also ensure to properly "sync" the filesystem before ejecting the card to ensure
all data has been written.

Notes:
 - Builds with bitbake accumulate in the deployment directory. You may want to
   delete older irrelevant images after repeated builds.

 - The first build will take a very long time because a lot of one-time house
   keeping and building has to happen. You want to have a powerful build machine.

 - SOURCE_THIS file has to be sourced when going back to build with a new shell.
