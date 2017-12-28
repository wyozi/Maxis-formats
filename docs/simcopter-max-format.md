.MAX maxis-archive geometry file format

## Introduction

Geometry data for models (buildings, helicopters, ground objects, etc) appears to be stored in the three sim3d?.max files.  Rough guess at a file format follows.


## Details

*All integers are Intel Little-Endian unless otherwise specified*

```
Header:
'DIRC'
int32: total size of DIRCchunk, incl. first 8 bytes (aka total file size)
int32: number of subchunks (typically 2)
'CMAP'
int32: pointer to CMAP data start (typically 0x1C)
'GEOM'
int32: pointer to GEOM data start (typically 829)

'CMAP' ("color map" - aka PALETTE)
int32: size of CMAP block, incl. 8 bytes 'CMAP'int32 header (typically 801)
int32: number of subblocks (just 1 here)
'CMAP' (subblock: the real palette starts here)
785 bytes: palette data
. int32: ???
. 3 x 3 bytes: Transparent colors?  "Magic" fullbright colors?
. int32: (offset 57): pointer to palette start, in file (or DIRC), typically 61
. 768 bytes: palette data, RGB, 3x256 with 8 bytes per component (24bits per entry)

'GEOM': geometry data start (typically offsett 829)
int32: size of geom block, incl. first 8 bytes
int32: num subblocks
int32: num geom units (subblocks - 1)
int32: offset to start of name table
int32: offset to start of model data

(geom name table)
17 bytes: null-terminated geom name
int32: offset into mesh table
int32: Flags?  1 = object, 143 = filename (sim3d1, etc)
int32: always 0
int32: Number of VERTICES
int32: always 0
int32: always 0
int32: Number of FACES
int32: ?? Possibly something to do with Texturing?
int32: Always 0

Duplicate GEOM table follows.
9xint32 in a row, just copies of the int32 set above, except without Flags and with a prepended ID

(geom object table)
Offsets are stored in the first int32 in the name table

'OBJX': marks an Object.  Items in the Names table point here.
int32: size.  Incl. header. But bugged: must add 12 bytes.
int16: QQ Number of Vertices in Vertex-array Block
int16: num FACES
int32: always 0
int32: ???? maybe a scale factor, a unique ID or a CRC or something
int32: always 0
88 bytes: null-terminated, null-padded object name
12 bytes:  JUNK.  This is where the bug comes in, above: somebody at Maxis calculated sizeof(Name) wrong.

Vertex-array Block
3 * (QQ) int32: Vertex array.  X, Y, Z coords, as signed int32 one after another.  First vertex in the list is the Origin.

YYY 'FACE' blocks (see object defs)
'FACE'
int32: size, incl. header
byte: num vertices
12 bytes: ???
vertices * int16s: Reference into Vertex table, above
8 more bytes per vertex: ???
```