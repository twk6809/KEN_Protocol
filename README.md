KEN PROTOCOL - README
=====================
By: Ken McCaughey  
On: 2024-12-22  
Ver 1.0.0  

<!-- In MarkDownd format. -->
<!-- Page breaks set for MarkText, US legal, with 10 top & bot.-->

Ken's Electronic Network (KEN) Protocol 

This serial communications protocol is intended for small micro-controller 
applications. This data link layer protocol specifies frame elements that 
define a variety of features. This protocol does not attempt to define all 
the rules in how they should or could be used. There are considerable 
permutations and it is up to the user to decide what fits a given 
application. Not all permutations that can be created are compatible with 
each other. The intent of this protocol is to have a set of tools and a 
structure to facilitate communications to meet a particular application. 
Only implement what is needed for a given application and tailor as needed. 
 
Universal compatibility is not the goal of this protocol. Ease of use and 
implementation is the goal.

--------------------------------------------------------------------------

## FEATURES

- Intended for small micro-controllers.
- Good for point-to-point messaging as well as small networks.
- Borrows ideas from MIDI, BiSync, and S.N.A.P. protocols.
- Features modular/scalable header elements.
  * Trades link overhead efficiency for modularity and parsing simplicity.
  * Header components can be mixed and matched as needed.
  * Allows for custom fields or features.
- Designed to be somewhat human readable with a hex viewer.
  * Most payload data is 7-bit (ASCII) with limited support for binary data.
  * Headers are nibble based.
    + First nibble denotes element function.
    + Second nibble contains variables.
- All control features in a header have the msb=1.
- Message sizes scalable based on complexity and options used.
  * Can be as short as 3-bytes.
  * Can optionally specify data length in header.
    + Nominal maximum data payload up to 14 ASCII bytes.
    + Can optionally extended data payload to 127 bytes.
  * Can have unlimited length if header data length control is not used.
    + Parsing data based on header flags only.
    + Checksum effectiveness decreases with messages longer than 256 bytes.
- Supports optional addressing.
  * Default simple addressing is up to 14 nodes.
  * Can optionally be extended to 128 nodes.
- Supports optional connection and error management.
  * Ack/nack.
  * Connect/disconnect.
  * Can use checksum or CRC error detection.
  * Supports message sequence numbering.
- Supports frame numbering with total frame count.
  * Used for messages spanning up to 127 frames.

--------------------------------------------------------------------------

## NOT INCLUDED
  
These are the things NOT included or specified in this protocol:

- Physical layer.
- Timing rules.
- Rules for managing connections and error management.
- Header size optimization.
- Byte (or bit) stuffing.
- CRC-32 checksum.
- Mechanism to auto assign addresses.
- Rules to enforce compatibility between different applications.
- Feature for universal plug-and-play like USB or TCP/IP.

--------------------------------------------------------------------------

## BASIC RULES

- Keep it simple.
  * Only used features that are needed.
  * Add complexity only as needed.
- All frame header elements are binary with msb=1.
- Frame payload data is ASCII with msb=0.
  * There is a limited provision for binary payload data.
- All frames have a required beginning and ending flags.
- Other than the two required frame flags, all other features are 
  optional and can be mixed and matched as needed.

<div style="page-break-after: always;"></div>

--------------------------------------------------------------------------

## FRAME FORMAT

Note that most significant byte is abbreviated as MSB (upper case) and 
most significant bit is msb (lower case). Similar for least significant 
byte or bits, LSB and lsb respectively. All header elements have MSB=1 to 
distinguish from data with MSB=0.  

All message frames begin with `0xFB` and end with `0xFE` flags. All other 
used header elements (`0x8n` to `0xEn`) are generally in numerical order 
before the data flag. The checksum, if used, is at the end prior to the end 
of message flag. Some header parameters may be followed by ASCII variables. 

Frame highlights:

- `++` Frame flags, most optional (hexadecimal)
- `**` Header control elements, optional (hexadecimal)
- `--` Flag or header ASCII component
- `==` Frame payload data
- `~~` Data covered by checksum

```
Flag [Header/Control Elements] Flag [Data] Flag [Check] Flag
++++ ************************* ++++ ====== ++++ ------- ++++
     ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

FB 8n 9n An Bn Cn Dn En FD [Data] FC [Check] FE
++ ** ** ** ** ** ** ** ++ ====== ++ ------- ++
 |  |  |  |  |  |  |  |  |  |      |  |       |
 |  |  |  |  |  |  |  |  |  |      |  |       +--end of frame flag *
 |  |  |  |  |  |  |  |  |  |      |  +--checksum
 |  |  |  |  |  |  |  |  |  |      +-----start check flag
 |  |  |  |  |  |  |  |  |  +--ASCII data
 |  |  |  |  |  |  |  |  +-----start of data flag #
 |  |  |  |  |  |  |  +--error control
 |  |  |  |  |  |  +-----data length
 |  |  |  |  |  +--------connection control
 |  |  |  |  +--to address
 |  |  |  +-----from address
 |  |  +--sequence number
 |  +-----checksum type (start of checksummed characters)
 +--beginning of frame flag *

 * required frame element
 # there are a different flags for different data types
```
