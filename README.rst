======================
GVIP Project Manifests
======================

GVIP project manifest files are used together with GVIP Yocto meta layer to
build the NXP Gateway Vehicle Integration Platform (GVIP).

First Time Setup
================

To get and build GVIP you need to have `repo` installed and its dependencies.
This only needs to be done once.

- Update the package manager (sudo apt update)

- Install `repo` tool dependencies:

   - Ubuntu 18.04 LTS or later
   - python 2.x - 2.6 or newer ($: sudo apt install python)
   - git 1.8.3 or newer ($: sudo apt install git)
   - curl ($: sudo apt install curl)

- Install `repo` tool::

   mkdir ~/bin
   curl http://commondatastorage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
   chmod a+x ~/bin/repo
   PATH=${PATH}:~/bin

Building GVIP
=============

The following steps will build a GVIP image based on NXP Auto Linux BSP image.

Note:
A Yocto build needs at least 50GB of free space and takes a lot of time (a few 
hours, depending on the system configuration). It is recommended to use a 
powerful system with many cores and a fast storage media (e.g., SSD).
The recommended RAM size is 8 GB.

Download the Yocto project environment
--------------------------------------

Get the latest GVIP manifest files and bring other required repositories::

  mkdir nxp-yocto-gvip
  cd nxp-yocto-gvip
  repo init -u https://source.codeaurora.org/external/autobsps32/gvip/gvip-manifests
  repo sync

Note: for a specific GVIP release or engineering build, please use the proper
branch (-b <branch>) and manifest file (-m <manifest>).

Note: to fetch internal development repositories use Bitbucket URL:
  repo init -u ssh://git@bitbucket.sw.nxp.com/gvip/gvip-manifests.git -b develop -m default-bitbucket.xml

Manifest files description:

 - default.xml -> fetch GVIP repositories from https://source.codeaurora.org
 - default-bitbucket.xml -> fetch internal development repositories from Bitbucket
 - alb.xml -> default NXP Auto Linux BSP without GVIP extensions

Setup the build environment
---------------------------

- Install all prerequisites before starting the Yocto build (first time only)::
   
   ./sources/meta-alb/scripts/host-prepare.sh

- Create a build directory and setup build environment::

   source nxp-setup-alb.sh -m <machine> -e "meta-aws meta-java"

  Currently, the only supported <machine> (NXP board) is: s32g274ardb2.

- Download PFE firmware from your nxp.com account and append the following lines 
  to file build_<machine>/conf/local.conf::

   DISTRO_FEATURES_append = " pfe"
   PFE_LOCAL_FIRMWARE_DIR = "<path to the local s32g_pfe_class.fw file>"

- Download GVIP CAN Gateway binary and append the following lines to file
  build_<machine>/conf/local.conf::
  
   DISTRO_FEATURES_append = " gvip-can-gw"
   GVIP_CAN_GW_DIR = "/home/nxa08592/yocto/bin"

Note: The features added above with DISTRO_FEATURES_append are optional and the
GVIP image can be built without those functionalities.

Note: To use internal development gvip repository add the following line in
build_<machine>/conf/local.conf::

  GVIP_URL = "git://bitbucket.sw.nxp.com/gvip/gvip.git;protocol=ssh"

Build the image
---------------

  bitbake fsl-image-gvip
  
Running the above command would be enough to completely build u-boot, kernel,
modules and a rootfs ready to be deployed. Look for a build result in
`build_<machine>/tmp/deploy/images/`.

Deploy the image
----------------

The file `<image-name>.sdcard` is a disk image with all necessary partitions and
contains the bootloader, kernel and rootfs. You can just low-level copy the data
on this file to the SD card device using dd as on the following command example::

  sudo dd if=fsl-image-gvip-s32g274ardb.sdcard of=/dev/<sd-device> bs=1M conv=fsync,notrunc status=progress && sync

Ensure that any partitions on the card are properly unmounted before writing
the card image, or you may have a corrupted card image in the end.
Also ensure to properly "sync" the filesystem before ejecting the card to ensure
all data has been written.

Notes:
 - Builds with bitbake accumulate in the deployment directory. You may want to
   delete older irrelevant images after repeated builds.

 - The very first build ever will take very long because a lot of one-time house 
   keeping and building has to happen. You want to have a powerful build machine.

 - SOURCE_THIS file has to be sourced when going back to build with a new shell.
