.. title: Analysing the length header of the AX100 radio
.. slug: analysing-the-length-header-ax100-radio
.. date: 2017-09-19 02:05:17 UTC+02:00
.. tags: 
.. category: sdrproject
.. link: 
.. description: 
.. type: text

This is a continuation from a `previous post about the AX100 radio
communication`_ where I concluded that the sync sequence in the
communication is not scrambled. This article is dedicated to discuss
the question whether the length parameter and the content of the sent
packet are also scrambled.

.. _previous post about the AX100 radio communication: https://olofsjodin.se/posts/finding-the-sync-on-a-gomspace-ax100-radio-data-stream-over-uhf/

A sequence of ping messages with incremented sizes ranged from 15 to
30 bytes has been sent from the radio. Other parameters such as
destination and type of error correcting code remains unchanged. My
hypothesis is that after the sync sequnce is followed by a length
parameter. The laboratory setup is the same as in the previous
article.

I found out that there is a field with the correct or similar value
when the message is descrambled thus the size of the sent packet is at
least the size of the CSP-header, payload and trailer. This is *not* a
real proof that the length is scrambled. It is however a notable
fact. After repeating the experiment with different packet lengths I
could verify that this field is indeed proportional to the length of
the packet. However, as shown below this length is only visible if the
preamble is shown as a sequence of fives instead of 'a' (represented
in hexadecimal).

Descrambled ping sequence 15 bytes::
  
  aaaa c985a8ef 1a 5308320000008101820283038404850586068766d3af461b8116
  aaaa b2616a3b c6 94c20c60000020406080a0c0e10121416181a1d8a20ec7b0d79f
  aaaa c985a8ef 1a 53083f8000008101820283038404850586068772a2eb3865e6de
  aaaa c985a8ef 1a 53083e800000810182028303840485058606872bb312f2af7402
  aaaa c985a8ef 1a 53083d800000810182028303840485058606870301dbeeb343a4
  aaaa b2616a3b c6 94c20f20000020406080a0c0e10121416181a1d68408891e745e
  aaaa b2616a3b c6 94c20ec0000020406080a0c0e10121416181a1dfa375c493f109
  aaaa b2616a3b c6 94c20e80000020406080a0c0e10121416181a1c9e70bb62155be
  ...
  5555 64c2d477 8d 2984190000004080c1014181c2024282c30343b369d7a30dc08
  5555 930b51de 34 a6106500000102030405060708090a0b0c0d0e9475e405bed3b
  5555 930b51de 34 a6106600000102030405060708090a0b0c0d0e7f84ad19a2279
  5555 64c2d477 8d 29841a0000004080c1014181c2024282c303439bdb7f7f11f72
  5555 64c2d477 8d 29841a4000004080c1014181c2024282c303438dafd1dd73834
  5555 64c2d477 8d 29841b0000004080c1014181c2024282c30343a30ae7147aa59
  5555 64c2d477 8d 29841b4000004080c1014181c2024282c30343b57e49b618d1f
  5555 64c2d477 8d 29841b8000004080c1014181c2024282c303438f821bf11fecf
  5555 64c2d477 8d 29841c0000004080c1014181c2024282c30343ab1f8f0729f9c

If the CSP-header is 4 bytes, Reed-Solomon 32 bytes and the packet is
15 bytes, we expect the length parameter to be a multiple of 51 bytes
or 0x33 bytes in hexadecimal. As we can see there is a field precisely
behind $930b51de$ which has the value 0x34. It is reasonable to
believe that the length size might include the length parameter itself
which is 1 bytes.

Descrambled ping sequence 20 bytes::

  aaaa b2616a3b c7 34c20ca0000020406080a0c0e10121416181a1c1e20222427beb
  aaaa b2616a3b c7 34c20c60000020406080a0c0e10121416181a1c1e202224272b2
  aaaa b2616a3b c7 34c20c00000020406080a0c0e10121416181a1c1e20222427606
  aaaa b2616a3b c7 34c20fa0000020406080a0c0e10121416181a1c1e20222426e0f
  aaaa b2616a3b c7 34c20f20000020406080a0c0e10121416181a1c1e2022242702e
  aaaa b2616a3b c7 34c20e60000020406080a0c0e10121416181a1c1e20222426bf5
  aaaa b2616a3b c7 34c20d40000020406080a0c0e10121416181a1c1e20222426ddd
  aaaa c985a8ef 1c d30834000000810182028303840485058606870788088909ea94
  aaaa b2616a3b c7 34c20ce0000020406080a0c0e10121416181a1c1e20222426c93
  aaaa b2616a3b c7 34c20fc0000020406080a0c0e10121416181a1c1e20222426abb
  aaaa c985a8ef 1c d3083e000000810182028303840485058606870788088909f70d
  ...
  5555 930b51de 39 a6106700000102
  5555 930b51de 39 a6106400000102
  5555 930b51de 39 a61062000001
  5555 930b51de 39 a6106100000102
  5555 64c2d477 8e 69841fc0000040
  5555 64c2d477 8e 69841f80000040
  5555 930b51de 39 a6107c00000102
  5555 930b51de 39 a6107b00000102
  5555 930b51de 39 a6107a00000102
  5555 64c2d477 8e 69841e00000040
  5555 64c2d477 8e 69841dc0000040
  5555 64c2d477 8e 69841d80000040
  5555 930b51de 39 a6107500000102
  5555 64c2d477 8e 69841d00000040
  
Here we can confirm that after increasing the size of the ping message
with 5 bytes the hexadecimal representation of the possible length
parameter do so as well. Thus the conclusion that this must be really
the part of the message responsible for representing the size of the
packet. Is it possible that the length parameter can be somehow
extracted from the other packets as well?

After analysing the messages I can also conclude that the message
right before the length parameter can be predicted as well, even if
the sync sequence is scrambled.

.. figure:: /images/length-header-analyse.png
   :align: center
   :width: 50%

   A figure representing an extraction of the binary part for the case
   with a 15 bytes ping message. The green, cyan and orange represents
   the equivalent part in both hexadecimal and binary.

The green part seems like it has "disappeared" as seen in the figure,
however, the orange part seems intact. Because of the property of the
orange part which seems constant no matter what the hexadecimal
representation is, then it might be possible to use this result as a
"sync sequence". This will be analysed more thorough in a later article.







