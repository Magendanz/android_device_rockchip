# android_device_rockchip
Device configuration for building TWRP on Rockchip reference boards 

## Build notes:
- To get started with OMNI sources to build TWRP, check out the minimal manifest repo [here](https://github.com/minimal-manifest-twrp/platform_manifest_twrp_omni).
- To save space, a shallow clone is recommended.
- These device configurations should be built with the **twrp-5.1** branch.
- Copy this project to /device/rockchip/ directory.
- The output is either ramdisk-recovery.img file or recovery.img, depending what your bootloader expects.
- The final image needs to be repacked as a Rockchip KRNL signed file.

On Rockchip devices, the offset to the bootloader message block in the /misc partition is 16384, rather than the usual zero or 2048.  Unfortunately, support for BOARD_RECOVERY_BLDRMSG_OFFSET is broken pending a pull request that I've submitted.  You may need to fix this yourself by adding the following lines to bootloader_message/Android.mk:

```
ifneq ($(BOARD_RECOVERY_BLDRMSG_OFFSET),)
    LOCAL_CFLAGS += -DBOARD_RECOVERY_BLDRMSG_OFFSET=$(BOARD_RECOVERY_BLDRMSG_OFFSET)
endif
```

Also, there are about a dozen or so command-line arguments (wipe_data, wipe_cache, update_package, etc.) defined in [Recovery.cpp](https://android.googlesource.com/platform/bootable/recovery/+/master/recovery.cpp), but Rockchip felt the need to add a custom one of their own, "wipe_all".  You'll need to make some tweaks to twrp.cpp after the get_args() call to support that.

Remember, the Rockchip RK33 bootloader calls recovery directly with this "wipe_all" command-line argument as part of the factory setup after flashing new firmware, so you really need to get this right or you'll end up with a TWRP boot loop.
