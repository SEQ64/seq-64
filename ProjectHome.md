SEQ64 is a full-featured editor for sequenced music in first-party Nintendo 64 games.

Click the Wiki tab for more information.
Click Source > Browse and find the "bin" folder to download, or compile from source.

Features:

  * Full MIDI import/export of sequences in Audioseq format
  * User-editable definition of Audioseq format
  * Edit sequence data directly (e.g. to add commands not supported by MIDI)
  * Edit and tag Audiobank instrument definitions
  * User-editable definition of Audiobank format
  * Move files in ROM while maintaining system integrity
  * Seamlessly recalculates CRC/CIC upon saving ROM
  * Supports all versions of games that use similar sequence formats
  * All ROM description data saved in human-editable format
  * Mature, comprehensible GUI
  * GPL licensed, cross-platform (Juce C++, full-speed on Windows, Mac, Linux)

Current project state:
  * MIDI export complete
  * MIDI import mostly complete, some bugs with call-based optimization
  * Editing which instrument set(s) are used with each sequence is complete
  * Audiobank parsing in progress

Available versions:
  * 32-bit Windows (compiled and tested on Windows 7)
  * 64-bit Windows (compiled and tested on Windows 7)
  * 64-bit Linux (compiled and tested on Ubuntu 14.04)

Games known to use sequenced music formats compatible with SEQ64:
  * Super Mario 64
  * Mario Kart 64
  * Yoshi's Story
  * Legend of Zelda: Ocarina of Time (`*`)
  * Legend of Zelda: Majora's Mask (`*`)
  * 1080 Snowboarding (+)
  * F-ZERO X (+)
  * Lylat Wars (+)
  * Pokemon Stadium (+)
  * Pokemon Stadium 2 (+)
  * Wave Race 64 (+)
(`*`: Relevant data is stored in compressed format; must be decompressed before use)
(+: Untested)