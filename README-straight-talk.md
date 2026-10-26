# android-16.0-straight-talk

This branch is BarkByte-AI's, for the straight-talk Pi 5 car image (`BarkByte-AI/straight-talk`,
`docs/2026-10-22-pi-image-build.md`, task 4). It is raspberry-vanilla's `a441c94` ("rpi5: update prebuilt kernel",
20 May 2026) with one file changed: `Image` is our rebuild of the same kernel with the SIM7600 USB modem drivers built
in, because nothing on the image loads modules. The dtbs, `overlays/` and `modules/` are `a441c94`'s, untouched; the
modules were built for the stock kernel and the image does not load them.

## The Image

- Release `6.12.85-gee5b110a00d1-v8`, the same as the stock kernel's; build string
  `(straight-talk@infra-aaos-kernel) #2 SMP PREEMPT Tue Oct 20 12:31:31 UTC 2026` (`uname -v`).
- sha256 `33a06a841831ce5d8029bd9b114e32a3b604d016791c7a36068384f30dbaf78a`, 51399168 bytes.
- It replaces the stock Image of `a441c94`: sha256 `0bc6a92af46f64746c497c2886669f199176a74b936f8df2db278714bc8016c1`,
  51329536 bytes (kleaf@build-host).

Identify the kernel by the Image's sha256, not by `strings Image | grep -m1 "Linux version"`: the first of the two
banners in our Image has an empty build number and no timestamp; the second matches `uname -v`.

## Inputs

The source is raspberry-vanilla `android_kernel_brcm_rpi` at `ee5b110a00d1` (`ee5b110a00d13ac5146108c4438478a7c7d7c07a`,
13 May 2026), the commit the stock Image was built from, configured with `android_rpi5_defconfig` and then the config
fragment `infra/aaos/kernel/usb-modem.fragment` in straight-talk. The fragment sets six options to `y`:
`USB_USBNET`, `USB_NET_CDCETHER`, `USB_NET_RNDIS_HOST` and `USB_NET_CDC_NCM` (the RNDIS data link as `usb0`, with ECM
and NCM), and `USB_SERIAL_WWAN` and `USB_SERIAL_OPTION` (the AT and NMEA ports as `ttyUSB*`). One more change is
accepted: `USB_NET_DRIVERS` m to y, the menu around the net drivers, which as a module caps every driver in it at m;
its other drivers stay modules. Apart from those, the config equals the stock kernel's `/proc/config.gz`.

`CONFIG_EXTRA_FIRMWARE` builds the Wi-Fi and Bluetooth firmware and `regulatory.db` into the Image from
raspberry-vanilla `proprietary_vendor_brcm` at `83f87c1733ef` (`83f87c1733ef992060f2996e94c7152f4521fd0d`), whose six
files are byte for byte those in the stock Image; that repository's later tip has a newer `regulatory.db`.

The build is `infra/aaos/kernel/build-rpi5.sh` in straight-talk (added in `f4a992c`). On a Mac it runs itself in
`ubuntu:24.04` as `linux/amd64` (Rosetta), because the toolchain the stock config records exists only for x86_64 hosts:
AOSP's `clang-r536225`, Rust `1.82.0.p1`, bindgen and kleaf's pahole 1.25 from the `main-kernel-2025` prebuilts, and
the NDK r26 sysroot that Kconfig's `CC_CAN_LINK` probe links against. Like kleaf, it builds from the commit's tree
without `.git` and writes the localversion `-gee5b110a00d1`, so the release cannot turn `-dirty`. The build's `.config` had
sha256 `4beace23ff3bea7c7cb2c8cb10f5d654dd99beaa87f8c702ea78703fcd8f9684`; a rebuild's Image will not hash the same as ours: the
timestamp and build number (the build directory persists in the container's volume, so the next build reads #3),
and the inputs the script does not pin, bindgen, pahole and the NDK sysroot from the moving `main-kernel-2025`
branch, the `ubuntu:24.04` image and its apt packages. The committed blob, not the script, is what the image build
takes.

## Catching up

raspberry-vanilla's `android-16.0` has moved on to `c29a85d` (30 September 2026, `6.12.93-gb07e73ed17bf-v8`); between
the two commits only `Image` and `modules/` changed. Do not merge their later commits here without a matching Image:
rebuild with `build-rpi5.sh` at kernel commit `b07e73ed17bf` with the same fragment (and the firmware commit that
matches that stock Image), check that the fragment's options stick and the rest of the config equals the new stock
config, then take their commit and replace `Image` on top of it.
