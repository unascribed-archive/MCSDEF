# Minecraft Skin Data Encoding Format
### [v1.0.0-pre1](http://semver.org)
This document is an attempt to devise a standard for encoding extra data into a Minecraft skin, in a way that is inherently compatible with vanilla, and all mods utilizing the format.

   The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
   "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
   document are to be interpreted as described in [RFC2119](https://tools.ietf.org/html/rfc2119).

## Abstract
### Rationale
A variety of Minecraft mods and services require data from the user, and users must re-enter this information on every server, or after every map reset. By encoding this information in the skin, the user can automatically set information that is readable by both the client and server, without any new packets or services.

### Preface
Minecraft (1.8) skins contain a total of 832 unused pixels of space. In an RGBA encoding, this totals out to 3.25KiB (3328 bytes) of space available for binary data. Pre-1.8 skins contain 480 unused pixels, totalling out to 1.875KiB (1920 bytes).

## Format
The MCSDEF format, when not inside a skin, is effectively a binary format. To help describe MCSDEF, first the inner format will be described, and then the skin encoding.
### Overall
All data files MUST be big-endian.
### Header
All data files MUST start with the string "SDEF" encoded in ASCII, followed by three unsigned bytes, consisting of the version of this document, in the order MAJOR, MINOR, PATCH (see [Semantic Versioning](http://semver.org/) for a description of this versioning scheme.) An unsigned byte MUST follow, consisting of 0, or the length of a following string. The contents of this string MUST be UTF-8, and should match the label on the version, such as "pre1". The string SHOULD NOT be NUL-terminated.

Following this string MUST be an octet, consisting of 8 bitflags. These bits mean the following:

 - **10000000**: Reserved
 - **01000000**: Reserved
 - **00100000**: Reserved
 - **00010000**: Reserved
 - **00001000**: Reserved
 - **00001000**: Reserved
 - **00000100**: Legacy Compressed
 - **00000010**: Compressed
 - **00000001**: Checksum

If the Checksum bitflag is set, the file MUST include a SHA1 checksum of the contents of the data section, immediately after the bitflag octet.

If the Compressed bitflag is set, the contents of the data section MUST be compressed with [Brotli](http://www.ietf.org/id/draft-alakuijala-brotli-04.txt).

If the Legacy Compressed bitflag is set, the contents of the data section MUST be compressed with DEFLATE.

The Compressed and Legacy Compressed bitflags MUST NOT both be set.

If the Compressed or Legacy Compressed bitflags are set and the Checksum bitflag is also set, the checksum MUST be derived from the compressed data rather than the uncompressed data.

Finally, there MUST be an unsigned 16-bit integer denoting the length of the Data section, in octets.

### Data
The Data section comes immediately after the Header section, and MUST consist of [NBT (Named Binary Tag)](http://wiki.vg/NBT) data.
The contents of the root Compound tag can be any kind of tag other than End, and the name MUST be a Java package name or Forge mod ID, followed by an at ("@") sign and a version in an arbitrary format. It is the responsibility of the mod or program parsing the data to understand the version string and the contents of the tag.

## Encoding
As mentioned in the preface, Minecraft skins contain unused pixels. Some mods with non-MCSDEF formats already use these pixels, but most only use a 8x8 pixel box in the top right of the skin. To help maintain compatiblity, MCSDEF implementations SHOULD NOT use this area of the skin.

Including this restriction, this leaves 768 pixels of space in a 1.8 skin, and 412 in a pre-1.8 skin. This gives us 3072 bytes to work with in a 1.8 skin, and 1648 in pre-1.8 skins.
The simplest way to encode binary data into an image, and the method used by MCSDEF, is to break up the data into 32-bit chunks and store the first byte in R, second in G, the third in B, and the fourth in A. If the size of the data is not a multiple of 4, the unused bytes SHOULD be set to 00000000, and readers will ignore it due to the length field in the header.

The pixels in the skin that are available for use are (in notation WIDTHxHEIGHT+X,Y):
 - 8x8+0,0 (the legacy area, usage discouraged)
 - 16x8+24,0
 - 8x8+56,0
 - 4x4+0,16
 - 8x4+12,16
 - 8x4+36,16
 - 4x4+52,16
 - 8x16+56,16
The pixels in the skin that are available for use in a 1.8 skin, in addition to the above, are:
 - 4x4+0,32
 - 8x4+12,32
 - 8x4+36,32
 - 4x4+52,32
 - 56x32+8,16
 - 4x4+0,48
 - 8x4+12,48
 - 8x4+28,48
 - 8x4+44,48
 - 4x4+60,48

To read or write the data in the skin, these regions MUST be addressed in the order above, from the smallest x, y value, left-to-right top-to-bottom, to the largest x, y value. In a programming language, this may look something like:

```java
for (int y = 0; y < 8; y++) {
	for (int x = 0; x < 8; x++) {
		/* do something with the pixel at x, y */
	}
}

```

It is RECOMMENDED to enable compression (or, as a fallback, legacy compression) and checksums when storing an MCSDEF file into a skin. Compression will allow a much larger amount of information to fit in the skin, and if the data is corrupted by an image editor or MCSDEF-unaware program, MCSDEF programs can fail gracefully rather than trying to load invalid data.

In an image editor, data stored in this format will look like noise.

## Implementations
There are currently no known finished implementations of MCSDEF. A reference implementation in Java is currently being worked on at [AesenV/MCSDEF-RI](https://github.com/AesenV/MCSDEF-RI).

## Copyright
This work is licensed under the Creative Commons Attribution-ShareAlike 4.0 International License.
To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/4.0/.

## Contact
If you have developed an MCSDEF implementation, please send an email to [Aesen](https://github.com/AesenV) with the address listed on his GitHub profile.

If you have questions related to the MCSDEF specification, or would like to submit a modification, submit an [issue](https://github.com/AesenV/MCSDEF/issues) or [pull request](https://github.com/AesenV/MCSDEF/pulls) to the MCSDEF GitHub repository.
