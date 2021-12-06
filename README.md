# Boot to Gecko (B2G)

Boot to Gecko aims to create a complete, standalone operating system for the open web.

You can read more about B2G here:

  http://wiki.mozilla.org/B2G
  
  https://developer.mozilla.org/en-US/docs/Mozilla/B2G_OS

Follow us on twitter: @Boot2Gecko

  http://twitter.com/Boot2Gecko

Join the Mozilla Platform mailing list:

  http://groups.google.com/group/mozilla.dev.platform

and talk to us on Matrix:

  https://chat.mozilla.org/#/room/#b2g:mozilla.org

Discuss with Developers:

  Discourse: https://discourse.mozilla-community.org/c/b2g-os-participation
  
# Build environment

As of today, only Linux hosts are supported.

You can read more about it on the following page :

  https://source.android.com/setup/build/initializing

# Building and running the android-10 emulator x86_64

1. Fetch the code: `REPO_INIT_FLAGS="--depth=1" ./config.sh emulator-10-x86_64`
2. Setup your environment to fetch the custom NDK: `export LOCAL_NDK_BASE_URL='ftp://ftp.kaiostech.com/ndk/android-ndk'`
3. Install Gecko dependencies: `cd gecko && ./mach bootstrap`, choose option 1 (Boot2Gecko).
4. Build: `./build.sh`
5. Run the emulator: `source build/envsetup.sh && lunch aosp_arm-userdebug && emulator -writable-system -selinux permissive`

# Buiding for devices

## Google Pixel 3a (sargo)

1. Fetch the code: `REPO_INIT_FLAGS="--depth=1" ./config.sh sargo`. This will also download the binary blobs for the device if they are not already present.
2. Setup your environment to fetch the custom NDK: `export LOCAL_NDK_BASE_URL='ftp://ftp.kaiostech.com/ndk/android-ndk'`
3. Install Gecko dependencies: `cd gecko && ./mach bootstrap`, choose option 1 (Boot2Gecko).
4. Build: `./build.sh`
5. Boot the Android system, go to settings, enable developer mode and enable OEM Unlock
6. Reboot into fastboot mode and issue
   - `fastboot flashing unlock`
   - `fastboot flashing unlock_critical`
7. Flash: `./flash_sargo.sh`

At boot time, you might need `adb shell setenforce 0` for B2G to boot (flash_sargo.sh does it).

## OnePlus X (onyx)

1. Fetch the code: `REPO_INIT_FLAGS="--depth=1" ./config.sh onyx`
2. Setup your environment to fetch the custom NDK: `export LOCAL_NDK_BASE_URL='ftp://ftp.kaiostech.com/ndk/android-ndk'`
3. Install Gecko dependencies: `cd gecko && ./mach bootstrap`, choose option 1 (Boot2Gecko).
4. Apply patch: `./patcher/patcher.sh`
5. Build: `./build.sh`
6. Boot the Android system, go to settings, enable developer mode and enable OEM Unlock
7. Reboot into fastboot mode
8. Flash:
   - `fastboot erase userdata`
   - `fastboot flash system $gonk_path/system.img`
   - `fastboot flash boot $gonk_path/boot.img`
  
If need to output a zip ROM file, you can use `./build.sh dist DIST_DIR=dist_output` instead of `./build.sh` in step 5.

## B2G Generic System Images (B2G-GSI) 
More [detail](https://github.com/phhusson/treble_experimentations/wiki) of which device is supported and which type for your device.  
1. Fetch the code: `REPO_INIT_FLAGS="--depth=1" ./config.sh b2g_gsi`
2. Setup your environment to fetch the custom NDK: `export LOCAL_NDK_BASE_URL='ftp://ftp.kaiostech.com/ndk/android-ndk'`
3. Install Gecko dependencies: `cd gecko && ./mach bootstrap`, choose option 1 (Boot2Gecko).
4. Build: `./build-gsi.sh [your_device_type_name] systemimage`

   type:
   - `gsi_arm64_ab` (arm64 partitionA/B)
   - `gsi_arm64_a` (arm64 partitionA)
   - `gsi_arm_ab` (arm partitionA/B)
   - `gsi_arm_a` (arm partitionA)

5. Flash: Follow the steps from [click it](https://source.android.com/setup/build/gsi#flashing-gsis) or use the `./create_gsi_dist.sh` script to create a self-contained archive with the image and flashing tools.

# Re-building your own NDK

Because it's using a different c++ namespace than the AOSP base, we can't use the prebuilt NDK from Google. If you can't use the one built by KaiOS, here are the steps to build your own:
1. Download the ndk source:
`repo init -u https://android.googlesource.com/platform/manifest -b ndk-r21d`
2. Get the code:
`repo sync -c --no-clone-bundle`
3. change `__ndk` to `__` in `external/libcxx/include/__config`:
```diff
diff --git a/include/__config b/include/__config
index 961acdb..b7ce8e3 100644
--- a/include/__config
+++ b/include/__config
@@ -127,7 +127,7 @@
 #define _LIBCPP_CONCAT(_LIBCPP_X,_LIBCPP_Y) _LIBCPP_CONCAT1(_LIBCPP_X,_LIBCPP_Y)
 
 #ifndef _LIBCPP_ABI_NAMESPACE
-# define _LIBCPP_ABI_NAMESPACE _LIBCPP_CONCAT(__ndk,_LIBCPP_ABI_VERSION)
+# define _LIBCPP_ABI_NAMESPACE _LIBCPP_CONCAT(__,_LIBCPP_ABI_VERSION)
 #endif
 
 #if __cplusplus < 201103L
```
4. Apply this change to `prebuilts/ndk`:
```diff
diff --git a/platform/sysroot/usr/include/android/log.h b/platform/sysroot/usr/include/android/log.h
index 512c7cd..b2c902b 100644
--- a/platform/sysroot/usr/include/android/log.h
+++ b/platform/sysroot/usr/include/android/log.h
@@ -131,36 +131,6 @@ int __android_log_vprint(int prio, const char* tag, const char* fmt, va_list ap)
 void __android_log_assert(const char* cond, const char* tag, const char* fmt, ...)
     __attribute__((__noreturn__)) __attribute__((__format__(printf, 3, 4)));
 
-/**
- * Identifies a specific log buffer for __android_log_buf_write()
- * and __android_log_buf_print().
- */
-typedef enum log_id {
-  LOG_ID_MIN = 0,
-
-  /** The main log buffer. This is the only log buffer available to apps. */
-  LOG_ID_MAIN = 0,
-  /** The radio log buffer. */
-  LOG_ID_RADIO = 1,
-  /** The event log buffer. */
-  LOG_ID_EVENTS = 2,
-  /** The system log buffer. */
-  LOG_ID_SYSTEM = 3,
-  /** The crash log buffer. */
-  LOG_ID_CRASH = 4,
-  /** The statistics log buffer. */
-  LOG_ID_STATS = 5,
-  /** The security log buffer. */
-  LOG_ID_SECURITY = 6,
-  /** The kernel log buffer. */
-  LOG_ID_KERNEL = 7,
-
-  LOG_ID_MAX,
-
-  /** Let the logging function choose the best log target. */
-  LOG_ID_DEFAULT = 0x7FFFFFFF
-} log_id_t;
-
 /**
  * Writes the constant string `text` to the log buffer `id`,
  * with priority `prio` and tag `tag`.
```
5. Build the ndk:
`python ndk/checkbuild.py --no-build-tests`
6. The build will end up in `out/dist`.
