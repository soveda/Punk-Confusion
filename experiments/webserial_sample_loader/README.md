# Punk Confusion WebSerial Sample Loader Experiment

This is an experimental branch-in-a-folder for loading Punk Confusion vocal
samples without rebuilding firmware or flashing a new UF2 for every sample set.

The intended flow is:

1. Flash the experimental firmware that matches your card:
   `punk_confusion_2mb.uf2` for normal 2 MB cards, or
   `punk_confusion_16mb.uf2` for 16 MB cards.
2. Open `web/index.html` in a Chromium-based browser.
3. Drop four short samples into the page.
4. Reset the card, connect with WebSerial, and upload during the 15 second
   loader window.
5. Restart the card and use the new shouts.

## First-Pass Design

- The same code builds both UF2 versions.
- The 2 MB build stores the sample bank in the top `1 MB` of program flash.
- The 16 MB build stores the sample bank in the top `14 MB` of program flash.
- The firmware prints its flash size and sample-bank size over WebSerial during
  the loader window, and the web UI can use that to match the selected limit.
- The uploaded bank contains a small header plus four 24 kHz 8-bit µ-law
  samples.
- Playback reads directly from flash, so normal RAM use remains small.
- Uploads are streamed into flash page-by-page, so the RP2040 never has to hold
  the full sample bank in RAM.
- µ-law is used because slicing and reverse playback stay simple: the firmware
  can jump to any byte offset and decode samples independently.
- If no valid uploaded bank is found, the firmware falls back to the current
  factory samples in `VocalSamples.h`.

## Useful Existing Workshop Code

- `releases/67_Fragments/src/fragments_24k.cpp`: compact flash erase/program
  pattern using `XIP_BASE`, `flash_range_erase`, and `flash_range_program`.
- `releases/41_blackbird/lib/flash_storage.cpp`: notes that flash writes stop
  XIP and must not race with other cores/audio work.
- `releases/73_VSS`: confirms 24 kHz µ-law flash-backed samples are a good fit
  for Workshop Computer sampler cards.

This is deliberately not the release firmware yet. It is a place to test the
upload protocol, flash layout, and playback behaviour safely.

## Build Outputs

The CMake project builds two targets from the same source files:

- `punk_confusion_2mb`: normal Workshop Computer/2 MB flash layout with a
  `1 MB` uploaded sample bank.
- `punk_confusion_16mb`: 16 MB flash layout with a `14 MB` uploaded sample
  bank.

Built test UF2s are kept in `uf2/`:

- `uf2/punk_confusion_2mb.uf2`
- `uf2/punk_confusion_16mb.uf2`

At 24 kHz 8-bit µ-law, the practical sample time is roughly 24,000 bytes per
second. That gives about 43 seconds total on the 2 MB build and about 10
minutes total on the 16 MB build, shared across all four shouts.
