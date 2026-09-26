# Relink archives

The TwyLapse Edge firmware statically links **Arduino-ESP32**, which is covered
by the **GNU Lesser General Public License, version 2.1 or later**. Section 6 of
that license requires that you be able to replace the library with your own
version and relink the program that uses it.

The ESP32 has no shared-library mechanism, so the option this project uses is
LGPL 2.1 section 6(a): the rest of the firmware is provided **as object code**,
together with the linker scripts, the exact link command and build
instructions. Each archive here corresponds to one published firmware image.

| File | Device | Firmware version |
|---|---|---|
| `tlp-edge-core-s3-<version>-relink.zip` | M5Stack CoreS3 | see the file name |
| `tlp-edge-stick-s3-<version>-relink.zip` | M5StickS3 | see the file name |

`manifest.json` lists every archive with its size and SHA-256.

Unpack an archive and read `BUILDING.md` inside it. In short:

```
python relink.py --arduino /path/to/your/arduino-esp32
```

This recompiles Arduino-ESP32 from the tree you point it at, puts the resulting
objects back into the supplied archives, links the firmware and produces a
flashable image.

No TwyLapse source code is included, and none is needed to relink.

The full license texts for every component of this product are shown in the
TwyLapse app, under *Menu → 著作権表示*.
