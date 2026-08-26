# Edge firmware images

Firmware for the HolyGrailController edge devices (M5Stack, ESP32-S3).
The phone app fetches `manifest.json`, picks the entry that matches the attached
device, downloads the image and writes it over USB.

## What an image is

Each `.bin` here is a **single merged image, written to offset 0**. It already
contains everything the device needs to boot:

| offset | contents |
|--------|----------|
| 0x0000 | second-stage bootloader |
| 0x8000 | partition table |
| 0xe000 | otadata (`boot_app0` — "start from app0") |
| 0x10000 | application |

Merging guarantees the bootloader, the partition table and the application are
always a matching set, and leaves the flasher with a single offset to handle.

**Writing an image erases the device's saved settings.** NVS lives at
0x9000–0xE000, inside the merged range, so Wi-Fi credentials, AP SSID/password,
device name, network mode and time zone are cleared. That is correct for a new
device — it comes up in AP mode with generated defaults, and the rest is set
through BLE provisioning. To update a device that is already provisioned, write
only the application part at 0x10000 instead.

## Picking the right image

Both edge models use an ESP32-S3, so the chip ID does not tell them apart.
**Flash size does**, and it can be read in download mode without writing
anything:

| flash size | model | image |
|-----------|-------|-------|
| 8MB | M5StickS3 | `hgc-edge-stick-s3.bin` |
| 16MB | M5Stack CoreS3 | `hgc-edge-core-s3.bin` |

## Verifying a download

`manifest.json` carries `size` and `sha256` for every image. Check both before
writing — a truncated download that reaches the flasher will brick the device
until it is re-flashed from a PC.

## How an image is produced

Built with PlatformIO, then merged with esptool:

```
esptool.py --chip esp32s3 merge_bin -o hgc-edge-stick-s3.bin \
  --flash_mode dio --flash_freq 80m --flash_size 8MB \
  0x0     bootloader.bin \
  0x8000  partitions.bin \
  0xe000  boot_app0.bin \
  0x10000 firmware.bin
```

`--flash_size` differs per model (8MB for StickS3, 16MB for CoreS3) and is baked
into the bootloader header, so the images are not interchangeable.
