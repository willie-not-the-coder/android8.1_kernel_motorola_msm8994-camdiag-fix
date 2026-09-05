# android8.1_kernel_motorola_msm8994-camdiag-fix[README.md](https://github.com/user-attachments/files/31866130/README.md)
# Motorola MSM8994 Kernel — Android 8.1 / kinzie

This tree is the Oreo kernel source used for the Motorola Moto X Force (kinzie / XT1580), Qualcomm MSM8994.

## Patches & Fixes

### Camera server diagnostics (CAMDIAG)

The camera bring-up patch adds `CAMDIAG` kernel logging to the Qualcomm camera sensor power/control path under:

- `drivers/media/platform/msm/camera_v2/sensor/io/msm_camera_dt_util.c`
  - `msm_camera_pinctrl_init()`
  - `msm_camera_power_up()`
- `drivers/media/platform/msm/camera_v2/sensor/io/msm_camera_io_util.c`
  - `msm_camera_request_gpio_table()`

The instrumentation reports camera-device/DT-node identity, GPIO counts and request-table presence, GPIO table contents and request results, pinctrl state acquisition, active-state selection, and function return status. The purpose is to diagnose the MSM8994 camera bring-up path and correlate kernel-side initialization with camera HAL/server failures on Android 8.1 Oreo.

The patch is diagnostic instrumentation; it does not replace the camera HAL or change application behavior.

## Build environment

Target:

- Device: Motorola Moto X Force XT1580 (kinzie)
- SoC: Qualcomm Snapdragon 810 / MSM8994
- Android target: 8.1 Oreo
- Kernel: Linux 3.10
- Architecture: ARM64
- Cross compiler: Android AArch64 GCC 4.9 (`aarch64-linux-android-`)
- Host build flag: `HOSTCFLAGS="-fcommon"`

Example environment:

```bash
cd /mnt/data/android8.1_kernel_motorola_msm8994
export ARCH=arm64
export CROSS_COMPILE=/mnt/data/toolchain/bin/aarch64-linux-android-
export PATH=/mnt/data/hosttools:$PATH
CC_REAL=/mnt/data/toolchain/bin/921d9846-a815-11e9-84ea-ff8bce1b90b6
```

Generate the kinzie configuration:

```bash
make O=out ARCH=arm64 CROSS_COMPILE="$CROSS_COMPILE" kinzie_defconfig
```

Build the kernel and DTBs:

```bash
make O=out ARCH=arm64 CROSS_COMPILE="$CROSS_COMPILE" \\
    HOSTCFLAGS="-fcommon" CC="$CC_REAL" -j"$(nproc)" Image.gz dtbs
```

The resulting kernel is:

```text
out/arch/arm64/boot/Image.gz
```

## DTB / QCDT packaging

The legacy Qualcomm QCDT image is assembled from the kinzie DTBs with `dtbTool` using a 2048-byte page size:

```bash
mkdir -p ./dtb_input
cp out/arch/arm64/boot/dts/qcom/msm8994-kinzie-*.dtb ./dtb_input/
dtbTool -o dt.img -s 2048 -p out/scripts/dtc/ ./dtb_input/
hexdump -C -n 4 dt.img
```

The expected QCDT magic is:

```text
51 43 44 54
```

For the Oreo boot image used in this reconstruction, the kernel payload is an uncompressed ARM64 `Image`. When starting from the build output, decompress `Image.gz` before replacing the boot-image kernel payload:

```bash
gzip -dc out/arch/arm64/boot/Image.gz > kernel
cp dt.img dt
```

Then repack the original Oreo boot image with a compatible `magiskboot` build (or an equivalent boot-image packer that preserves the original header/ramdisk layout):

```bash
magiskboot unpack boot.img
# replace the extracted kernel/dt payloads as appropriate
magiskboot repack boot.img new-boot.img
```

Before flashing, verify the final image structure and checksums. RAM-only testing is preferred first:

```bash
fastboot boot new-boot.img
```

## CAMDIAG runtime verification

Kernel diagnostics:

```bash
adb shell 'dmesg -w' 2>&1 | grep -Ei --line-buffered \
  'CAMDIAG|msm_camera_dt_util|msm_camera_io_util|msm_camera|camera_v4l2_open|mot_imx230|msm_sensor'
```

Android/logcat diagnostics:

```bash
adb logcat -v threadtime 2>&1 | grep -Ei --line-buffered \
  'CAMDIAG|mm-camera-intf|QCamera|QCamera2HWI|QCameraStream|QCameraChannel|CamDev@1.0-impl'
```

Crash verification:

```bash
adb logcat -b crash -d
adb shell dmesg | grep -Ei 'NEW_SESSION|msm_post_event|camera_v4l2_open|Connection timed out|errno 110|SIGSEGV|SIGILL'
```
