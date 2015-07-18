# Audioseq File Format #

Audioseq is a proprietary format for sequenced music commands, developed by Nintendo in the mid-1990s for their N64 development kits. With minor variations between games, it was used as the primary music format in several first-party N64 games, including Super Mario 64, Mario Kart 64, and the two Zelda games.

## Credits ##

I got started on reversing the Audioseq format based on the work of messiaen

https://sites.google.com/site/messiaen64/mario-64-sequenced-music-specification

and Deathbasket

https://sites.google.com/site/deathbasketslair/zelda/ocarina-of-time/sequenced-music-format

Without their work, I would have not been able to get anywhere with this project; they figured out most of the basic commands.

## Overall Description ##

In games that use this format, there is a file in the ROM (called "Audioseq" in the Ocarina of Time Debug ROM, which is where the name came from) containing the data for all the sequences in the game. This file is made up of several sequence files, one per song; and somewhere (either at the beginning of Audioseq, or elsewhere, depending on the game), there is an index of all the sequences with pointers to the start of each sequence and its length, so the single sequence can be copied into RAM at load time. The pointers in the Audioseq Index are relative to the start of Audioseq; the pointers within the sequence files themselves are relative to the individual sequence file (unless noted otherwise, in which case they will be relative to the address of the command after the current command).

A sequence file is in a stream-based format (as opposed to a struct-based format), made of commands that are read one after another (like a MIDI file). The commands fall into three categories:
  * Music-related commands (e.g. track note, channel volume, pitch bend, etc.)
  * Timing commands (wait a certain amount of real time before parsing the next command)
  * Control commands (e.g. move playback around the sequence file in various ways)

Like in a MIDI file, a sequence has 16 channels (some of which may be unused). Each channel has up to 4 note layers, each of which is monophonic. (The sequence format itself may seem to support 8 or 16 note layers, but I have never seen more than 4 used, and I have not tested setting up more than 4.) The only commands that apply to the note layers are note commands and transpose commands; commands like volume and pitch bend apply to channels, and commands like tempo and master volume apply to the whole sequence.

The sequence begins with a sequence header, which contains commands that apply to the whole sequence. Among these are pointers to channel headers. Channel headers likewise contain channel commands, and pointers to data for the note layers (usually called "tracks") for that channel. To use an analog analogy, the playback engine has a play head for the sequence header, one for each channel, and one for each active note layer. All of these are running simultaneously, and run arbitrarily fast (compared to the music tempo); however, when any particular one encounters a timing command, it waits (in real time) before parsing the next command. Thus, in a simple sequence, the total length of the timestamps in each of the note layers will be equal to each other and to the total time in the channel, since they're all running simultaneously.

Most sequences are organized into temporal sections. One section is the sequence header having pointers to each of the channels, and then a timestamp equal in length to the time-lengths of each of the channels (and their tracks). Thus, all of the headers and tracks are finished at the same time. Then, the section has pointers to new channel headers; this in effect moves the channels' play heads to those new positions, and the song continues (seamlessly). One example for why sections are used is that there can be a section for the introduction to the song, and then a section for the part that is looped. In this case, the second section (after its timestamp) will have a control command to tell the sequence header's play head to jump back to the beginning of the second section, which re-initializes the channels to play that section again (thus looping the section). Some games use more complicated sets of commands to jump to different sections depending on certain initial conditions.

Timing is measured in ticks, and there are 0x18 (24) ticks per quarter note. Commands in the sequence header set the tempo in BPM (quarter notes per minute).

Some commands are variable-length; however, the format is unlike that of MIDI. Two formats are known:
  * Up to 1 byte (i.e. 0 or 1 bytes): if the next byte is <= 0x7F, it is read and used as the value (7 bits data). Otherwise, the value is assumed to be zero (0 bits data), and no extra byte is read (the next byte is the first byte of the next command).
  * Up to 2 bytes (i.e. 1 or 2 bytes): One byte is read. If it is <= 0x7F, it is used as the value (7 bits data). Otherwise, 0x80 is subtracted from it, it is shifted left one byte, and the following byte is read and or'd with it (15 bits data in total). For example, counting up from 126: 0x7E, 0x7F, 0x8080, 0x8081, ... 0x80FF, 0x8100, ... 0x8FFF, 0x9000, ... 0xFFFF

Several of the control commands are jump/branch commands, which move the playback head that executes them to another position. However, of special note are the call and loop commands. A call works like in assembly language; the play head jumps to another position, but when it reaches the end of data there, it returns to the command after the call (the return address is saved on a stack 4 entries tall). A loop consists of a loop start and loop end command, with data in between; the data is repeated as many times as are specified in the loop count parameter of the loop start command (there is no offset; a count of 4 will play the data 4 times). This also uses the stack: the address of the command after the loop start is pushed on the stack, as well as the current count value (on a separate stack also of length 4). When the loop end command is reached, the loop-start-address is peeked from the stack and the count is decremented; if the count is zero, the return address is popped.

## Commands ##

Keep in mind that RomDesc files contain all this information in a (semi-)human-readable format, and can be loaded and edited with seq64.

Key:
  * Cmd: The lowest valid byte for the command (inclusive).
  * End: If the command has a range, the highest valid byte for the command (inclusive)
  * DataSrc: where does this parameter's value come from? Offset: the actual command minus the lowest valid command byte. Fixed: a value in a fixed number of bytes. Variable: a variable length quantity as above.
  * DataLen: For Fixed or Variable parameters, the data length.
  * Seq, Chn, Trk: Whether this command is valid in sequence header, channel header, or track data.
  * Q: A one-byte variable in the RAM associated with the sequence control/state, used as a temporary "register" for math commands. Think "aQmulator". ;)
  * C: The value of the command offset. (E.g. command is 90-9F, we have 93, offset is 3)
  * `var[]`: An array of 8 (or maybe 16?) bytes in the RAM associated with the sequence. The particular functions of each are unknown, but from looking at sequences, sometimes a sequence reads from its contents to determine what section to play, or writes back to it when it has gotten to certain points.
  * Absolute means relative to the beginning of the sequence. Relative means relative to the address of the command after the current one.

### Control flow commands ###

|Cmd|End|DataSrc|DataLen|Description|Seq|Chn|Trk|
|:--|:--|:------|:------|:----------|:--|:--|:--|
|F2 |-- |       |       |Rel Branch on Q < 0|X  |X  |X  |
|   |   |Fixed  |1      |--Relative Address|   |   |   |
|F3 |-- |       |       |Rel Branch on Q == 0|X  |X  |X  |
|   |   |Fixed  |1      |--Relative Address|   |   |   |
|F4 |-- |       |       |Rel Branch always|X  |X  |X  |
|   |   |Fixed  |1      |--Relative Address|   |   |   |
|F5 |-- |       |       |Abs Branch on Q >= 0|X  |X  |X  |
|   |   |Fixed  |2      |--Absolute Address|   |   |   |
|F6 |-- |       |       |Abs Branch always to 0 (beginning of sequence)|X  |X  |X  |
|F7 |-- |       |       |Loop End   |X  |X  |X  |
|F8 |-- |       |       |Loop Start |X  |X  |X  |
|   |   |Fixed  |1      |--Loop Count|   |   |   |
|F9 |-- |       |       |Abs Branch on Q < 0|X  |X  |X  |
|   |   |Fixed  |2      |--Absolute Address|   |   |   |
|FA |-- |       |       |Abs Branch on Q == 0|X  |X  |X  |
|   |   |Fixed  |2      |--Absolute Address|   |   |   |
|FB |-- |       |       |Abs Branch always|X  |X  |X  |
|   |   |Fixed  |2      |--Absolute Address|   |   |   |
|FC |-- |       |       |Abs Call   |X  |X  |X  |
|   |   |Fixed  |2      |--Absolute Address|   |   |   |
|FF |-- |       |       |End of Data / Return|X  |X  |X  |

### Control data commands ###

|Cmd|End|DataSrc|DataLen|Description|Seq|Chn|Trk|
|:--|:--|:------|:------|:----------|:--|:--|:--|
|00 |0F |       |       |Load an unknown bit from channel C's control RAM to Q|X  |   |   |
|50 |5F |       |       |Q -= `var[C]`|X  |   |   |
|70 |7F |       |       |`var[C]` = Q|X  |   |   |
|80 |8F |       |       |Q = `var[C]`; if(C < 2) `var[C]` = -1;|X  |   |   |
|C8 |-- |       |       |Q -= val   |X  |   |   |
|   |   |Fixed  |1      |--val (unsigned)|   |   |   |
|C9 |-- |       |       |Q &= val   |X  |   |   |
|   |   |Fixed  |1      |--val      |   |   |   |
|CC |-- |       |       |Q = val    |X  |   |   |
|   |   |Fixed  |1      |--val      |   |   |   |

Several other commands exist but are not yet figured out.






|D3|--| | |Sequence Format|X| | |
|:-|:-|:|:|:--------------|:|:|:|
|  |  |Fixed|1|Value (unknown meaning)| | | |