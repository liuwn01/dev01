---
title: "[MS-ABS]: Run Encoding"
description: "As described in the previous section, when a run of characters is encoded, a length and offset are stored in a minimum of"
---

# 5.3 Run Encoding

<p>As described in the previous section, when a run of
characters is encoded, a length and offset are stored in a minimum of 2 bytes
and a maximum of 6. This encoding works by storing the offset in the high 13
bits of the first two bytes and the length in the low 3 bits. Because the
compression algorithm does not generate runs shorter than 3 bytes, the length
is biased by -3. </p>

<p>The first 2 bytes are treated as an unsigned 16-bit integer,
with the high 13 bits being the offset and the low 3 bits being the length. The
offset has been biased by -1, so in order for the number to be useful, 1 must
be added back to it. The offset is a backwards offset from the current position
in the output buffer. So after adjusting for the -1 bias, an offset of zero in
the high 13 bits would be 1, and that value (1) would then be subtracted from
the current position in the output buffer to get the location of the previous
character in the buffer that starts the run. </p>

<p>The length field is biased by -3, because no runs of length
less than 3 can be emitted by the compression algorithm. So a value of 0 in the
low 3 bits is really a length of 3. This bias makes it possible to encode
lengths of 3 to 10 (0 through 7) in 3 bits. Because the length will often be
greater than 10 bits, there is an additional mechanism: If all 3 bits are set
(0x7), then the next byte is examined. The first time, the low 4 bits of that
byte are extracted as the length, and the location of the byte is stored. This
4-bit length is biased by -10 (-(3+7)), so it is now possible to represent
lengths of 10 through 25 using the three-byte notation. In fact, only two and a
half bytes are available, because each run of length 10 to 25 uses either the
low or high four bits of the byte. Again, to accommodate lengths greater than
25, there is another escape mechanism: If all 4 bits are 1 (0xF), then the next
byte is also examined. If the next byte is not equal to 255 (0xFF), then that
is the length, biased by -25 (-(3+7+15)), so it is possible to encode lengths
of 25 through 279. Finally, if the fourth byte is 255 (0xFF), then the next two
bytes contain the length, biased by -3. The following section contains pseudo
code to demonstrate this encoding algorithm. The decompression pseudo code in
the section after that shows how to decode the encoded offset and length.</p>


                