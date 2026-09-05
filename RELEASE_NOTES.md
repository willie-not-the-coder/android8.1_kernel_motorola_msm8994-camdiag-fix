# v8.1-kinzie-camdiag-fix

Motorola Moto X Force XT1580 (kinzie), Android 8.1 Oreo MSM8994 kernel CAMDIAG diagnostic patch.

## Source

Commit message:
`kinzie: Fix camera server crashes via CAMDIAG patch for Android 8.1 Oreo`

The patch instruments the Qualcomm camera sensor power/control path in:

- `drivers/media/platform/msm/camera_v2/sensor/io/msm_camera_dt_util.c`
- `drivers/media/platform/msm/camera_v2/sensor/io/msm_camera_io_util.c`

## Build

Android GCC 4.9 AArch64, `ARCH=arm64`, `HOSTCFLAGS="-fcommon"`.

```bash
make O=out ARCH=arm64 CROSS_COMPILE="$CROSS_COMPILE" HOSTCFLAGS="-fcommon" CC="$CC_REAL" -j"$(nproc)" Image.gz dtbs
```

## Artifacts

| File | Size (bytes) | SHA-256 |
|---|---:|---|
| `Image.gz` | 8490479 | `4341ad22d61f874eab89ef30054c19eaa82affb041103328fac806482b16a00` |
| `dt.img` | 2695168 | `cbf7880aea19e1614e8422347dfa23d3512d75d29646dcdb0661bbb7e2bb59da` |
| `new-boot.img` | 30457856 | `955901d97e2a88f654f7e5a524607c37be7e21c327aa0bba8597e69f65f5f6f0` |

`dt.img` has Qualcomm QCDT magic `51 43 44 54` (version 3).

The final boot payload uses the decompressed ARM64 `Image` form because the original Oreo boot image stores an uncompressed kernel payload.
