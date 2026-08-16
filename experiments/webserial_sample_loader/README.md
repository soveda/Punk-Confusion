# Punk Confusion WebSerial Sample Loader Experiment

This is an experimental branch-in-a-folder for loading Punk Confusion vocal
samples without rebuilding firmware or flashing a new UF2 for every sample set.

The intended flow is:

1. Flash this experimental firmware once.
2. Open `web/index.html` in a Chromium-based browser.
3. Drop four short samples into the page.
4. Reset the card, connect with WebSerial, and upload during the 15 second
   loader window.
5. Restart the card and use the new shouts.

## First-Pass Design

- The sample bank is stored in the top `1 MB` of program flash.
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
