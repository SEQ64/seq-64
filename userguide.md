# Files Tab #

Load your ROM and the ROM Description File (RomDesc) with the appropriate menus. If you are starting on a ROM from scratch, I recommend you copy the RomDesc from another game, so you have a starting point for the sequence format.

If you define a valid address to a file table in the ROM (not all games have this), the file table contents will appear in the Master File Table box on the Files tab. You can rename the files and tell seq64 what type these files are. All file types that are not audio-related are just for your convenience. All this information (but not the file table itself!) is stored in the RomDesc.

The Known Files box stores references to all the files seq64 knows how to deal with. You can get files in this box by labeling existing files from the file table, or by manually adding them as Known Files.

Click on an Index file in Known Files to load the contents of the index into the Index box. If not already defined, you may have to set the index format using the option buttons at the top of the Index box. All Index entries can be named; this information is stored in the RomDesc file. Make sure to save it often!

If the index being shown is the Audioseq Index, and you select a sequence in the index, click the Load Entry button to load the sequence from ROM into memory for editing/export. The currently loaded sequence will show up on the Audioseq tab.

# Audioseq Tab #

## Command Editor ##

This lets you define the Audioseq format. This information is stored in the RomDesc.
  * Name: User-defined name for the command.
  * Command: Lowest and highest values for the command's type (first) byte. For instance an absolute pointer to a channel header is often 90-9F. If the command only has one possible value for its type, leave the second box blank.
  * Action: How the MIDI engine will react to this command.
  * Valid in: Whether this command is recognized in sequence header data, channel header data, and/or track data.

### Parameters ###

This lets you specify the data that comes after (or in!) the command's type byte, to make the whole command. The box just lists all the parameters.
  * Name: User-defined name for the parameter.
  * Data source:
  1. Cmd Offset: The difference between the command's type byte and the lowest possible value. For instance, a command with type byte 9A might be an absolute pointer to channel header (9x) on channel 10 (xA).
  1. Fixed: Data value for this parameter is a fixed number of bytes, as entered in the length box.
  1. Variable: Data value for this parameter is a variable length (NOT the same format as MIDI variable length quantities). There are two supported types, with up to 1 or up to 2 bytes of data. The value of up to 1 byte is read if it's <= 7F, and ignored if the value is >= 80 (the next command). The value of up to 2 bytes is read as either 1 or 2 bytes; 1 byte if the byte is <= 7F, and 2 bytes if the first byte is 80, but mask out the 80 for a 15-bit result. So for instance values 00 - 7F are represented as one byte of that value, then a value of 80 is represented as 8080, a value of 100 is 8100, and a value of 1000 is 9000. I plan to explain this more fully in the page about the Audioseq format.
  * Meaning: How the MIDI engine treats this parameter.
  * Add and Multiply: Support is deprecated; it turns out that the commands tend to have ranges that are very similar to MIDI, and they don't vary between games very much. Just leave Add at 0 and Multiply at 1.

## Loaded Sequence ##

This is the parsed sequence that is currently loaded into memory.

The Sections box shows the sections of the sequence, based on where pointers point to. There is no index of these sections; so if you delete a pointer to a section, it stops being listed here! But the data is still there.

Select a section to show the sequence data in the Commands box. Select a command type from the combo box and click Add to add it before the selected command. Select a command and click Delete to delete it. Both of these actions update pointers throughout the sequence to keep it correct.

Select a command, select one of its parameters, and type in the Value box to edit the parameter. If it's variable length, and you change the value such that the length would change, it inserts or removes the space and fixes all the pointers in the sequence.

## MIDI File ##

MIDI export is working, but import is not. The controls should be self-explanatory.