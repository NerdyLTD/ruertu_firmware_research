# Ruertu RTEB103C - public stock firmware research
Prepared from observations recorded 29 October 2026. This is research reference
material, NOT a flashable recovery kit, unlock guide, or working custom ROM. However,
within the files in this repo, you'll find "Ruertu_Stock_Firmware_Download_Helper.7z,"
which contains instructions in Markdown format, along with a PowerShell helper script
that will assist you in downloading and extracting the official firmware directly from
Ruertu's servers (which for some reason are not at ruertu.com, but I have no idea why.)

*************************************************************************************
ALWAYS VERIFY THE SAFETY OF A SCRIPT BEFORE YOU RUN IT! ASSUME NOTHING IS SAFE!
*************************************************************************************

## Scope and provenance
One Ruertu RTEB103C running Android 11 was inspected. Vendor properties:
ro.build.versionCode=HKC; ro.build.versionName=RTEB103CPR4.6.420231122.
The Settings APK exposed the OS-check implementation; Launcher static analysis
encountered a baiduprotect-associated wrapper. No firmware was installed or
flashed and no bootloader unlocking or rooting was performed.

A local encrypted request to the configured legacy endpoint returned HTTP 200
and offered the same apparent 4.6.4 / 20231122 release, not a demonstrated newer
version. Endpoint: http://ereader.xrztech.com:8090/json/osupgrade.action
The request uses sn, osName, osVersion; RSA/ECB/PKCS1Padding and Base64. The native
wrappers reference separate request-public and response-private keys. No key
material or actual serial is included. The Settings serial helper reads
vendor.gsm.serial, but the original OS-check caller was not recovered.

## Original download
https://bj.bcebos.com/bos-s3/hkc/RT_EB_103C_P_R_4.6.4_20231122-OTA-FULL.zip
Length: 1274940004 bytes. MD5: 752949be4fd7082f164be7af6271718a.
The archive builder rechecked that size/MD5 and computed the hashes included here.
Availability at future dates is not guaranteed. Firmware binaries are not bundled.

## Package and extraction
A/B OTA, payload version 2, 4096-byte blocks. Five ZIP entries: Android metadata,
care_map.pb, payload.bin, payload_properties.txt and an OTA certificate.
pre-device=k65v1_64_bsp; Android SDK 30; security patch 2021-01-05.
post-build=alps/full_k65v1_64_bsp/k65v1_64_bsp:11/RP1A.200720.011/mp1V891:user/release-keys
Fifteen replacement-only partitions, no old-image dependency:
boot, system, vendor, product, preloader, md1img, spmfw, sspm, gz, lk, dtbo, tee,
vbmeta, vbmeta_system, vbmeta_vendor. Total target bytes: 2916331520.
Extractor: ssut/payload-dumper-go 2.1.0, default checks left enabled; all images
completed with no errors reported. Command: payload-dumper-go.exe -o <out> <OTA-ZIP>
Project: https://github.com/ssut/payload-dumper-go

## Boot and security observations
Boot header v2; kernel 11863430 bytes; ramdisk 8608704 bytes; page size 2048.
Command line: bootopt=64S3,32N2,64N2 androidboot.selinux=permissive buildvariant=user
All three vbmeta headers: SHA256_RSA2048, rollback index/location 0, flags 0,
required libavb 1.0, release string avbtool 1.1.0. Signatures were not independently
verified. The unit reported SELinux Permissive, non-root ADB uid=2000, green
Verified Boot and a locked bootloader. Permissive is not root or an AVB bypass.

## Useful applications and limits
Useful for stock-image comparison, APK/configuration recovery, boot/init and
vendor/display analysis, and device-specific custom-firmware/recovery research.
Stock boot/ramdisk, vendor dependencies and DTBO may inform future development;
this is not kernel source, a device tree, or complete build configuration.
No unlock, root install, GSI/custom ROM boot, custom recovery or brick restoration
was demonstrated. Shared MediaTek/Bigme identifiers do not prove compatibility.
Missing: userdata, device-unique calibration/NVRAM, partition table and ready-made
super.img. Logical partition handling still needs device-specific investigation.
Do not flash preloader, lk, tee, vbmeta or patched boot based on this report.

## Integrity and authenticity
MD5/size agreement and SHA-256 inventories establish file identity, not official
endorsement or cryptographic trust. Discovery used plain HTTP; download used
HTTPS. Independent OTA/payload signature and certificate trust validation were
not documented. Public hash rows are freshly computed local values, not an
assertion of an independent comparison to payload hashes. Missing image names
are recorded separately. No proprietary firmware/APKs or private key material are
included; redistribution rights for those binaries have not been established.

## Publication
This PUBLIC folder contains generated Markdown documentation, fixed-name firmware
hashes, a checksum manifest, an omissions report and a restrictive .gitignore.
It never copies your documents, source code, conversations, logs or binary files.
Review the folder before uploading. No license is asserted for upstream firmware
or tools. Do not publish a sibling PRIVATE archive. Git history can preserve
accidentally committed files even after deletion; prevention matters.

References:
https://android.googlesource.com/platform/system/update_engine/+/HEAD/update_metadata.proto
https://source.android.com/docs/core/architecture/bootloader/boot-image-header
https://android.googlesource.com/platform/external/avb/+/master/libavb/avb_vbmeta_image.h
