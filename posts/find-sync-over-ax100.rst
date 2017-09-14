.. title: Finding the sync on a GomSpace AX100 radio data stream over UHF
.. slug: finding-the-sync-on-a-gomspace-ax100-radio-data-stream-over-uhf
.. date: 2017-09-13 17:14:06 UTC+02:00
.. tags: matlab, sdr, cubesat, seam
.. category: sdrproject
.. link:
.. description:
.. type: text

Currently I'm writing a program in Matlab that is taking an incoming packet,
read the length parameter, try to reverse bit errors by using the Reed-Solomon
algorithm on the packet and, finally, extract the CubeSat Space Protocol (which
is from now on called *CSP*) header and see where it should go. The `code`_ is
somewhat partly written, however, I've noticed that the CSP header don't match
with what I know about the setup. For example is source and destination address
somewhat wrong hence my suspicion about some of my assumption I made about the
incoming data stream, especially the sync sequence. The purpose of this article
is to give a brief summary about how I found the sync sequence.

I've generated testcases with a python script I made called
`testcases_uhf.py`_. CSP-term, a program to connect to the satellite radio
AX100, runs the generated testcases while the satellite radio is connected to
our SDR device hence the need for a script that can effectively generate
different test cases.  The limitation is though that a script cannot be longer
than 99 lines, the reason for this is unknown.

.. _testcases_uhf.py: https://github.com/medik/AX100-Tools/blob/master/testcases_uhf/testcases_uhf.py

The following data is sent from the device:

- Three ping packets with size 100 to address 1 to 4
- Three ping packets with size 10 to address 1 to 4
- Three packets with the message FF00FFFFFFFAAAAAFFFF00000010000000 in hexadecimal

On the recieving side I have configured LabView Communication to record both the
scrambled and the descrambled data. Ah, yes, the AX100 has a built in scrambler.
GomSpace has reused parts from other projects, including the `AX5043`_ which is
made by Onsemi. As seen in the `programming manual`_ at page page 37, the
scrambling polynomial is $1+x^{12}+x^{17}$ (thank you M. Gruber for finding
this!) hence the implementation of a descrambler in LabView
Communication. Furthermore, we also know *when* it turns on. In the same manual
on page 21 under the headline "Modulation & Framing" we can see parameters in
the AX100 that are otherwise undocumented. By flipping the mode from RAW to SYNC
we can confirm that the scrambler is indeed turned on thus the parameter is
responsible for scrambling on the AX100.

M. Gruber wrote a program in Python that searches a data stream with a search pattern. I have though slightly
modified it to omit the first part of the preamble. By searching for the
preamble sequence of 50 bytes with a pattern of $10101010...$ we acquired the
following results.

.. figure:: /images/ax100-undescrambled-output-sync.png
   :align: center
   :width: 50%

   The figure represents the data packets which are sent RAW i.e. not scrambled.

.. figure:: /images/ax100-descrambled-output-sync.png
   :align: center
   :width: 50%

   The data packets sent in SYNC mode descrambled.

As we can see in the two previous figures, the zeroes becomes either b261a3b or
c985a8ef when SYNC mode is toggled. However, it is not known whether the sync is
scrambled or not nor if it is important to know that. What would the sync
sequence be if it is not scrambled?

.. figure:: /images/ax100-undescrambled-output-sync-2.png
   :align: center
   :width: 50%

   The data packets sent in SYNC mode scrambled.

Here we see that the sync sequence are either 6cf8a828 or 9b3e2a0a. Okay, what
is going on here? Are there two different sync sequences?

================================ ========
Binary                           Hex
================================ ========
10011011001111100010101000001010 b261a3b
11001001100001011010100011101111 c985a8ef

01101100111110001010100000101000 6cf8a828
10011011001111100010101000001010 9b3e2a0a
================================ ========

b261a3b seems not like a permutation of c985a8ef, however, 6cf8a828 does look
like a permutation of 9b3e2a0a. The only difference is the two first bits in the
latter sync sequence which looks like it is a part of the preamble. If the sync
sequence is scrambled, then the first two values in the table should be similar
hence the conclusion that the sync sequence *isn't scrambled*. Okay, if it isn't
scrambled; does that mean the first two bits after 9b3e2a0a is 00? Let's find
out: 1 from hex to binary is 0001. Fortunately this is true for every 9b3e2a0a I
find.

This was a snag, but there is a lesson to be learned. Never, ever, assume
anything. There is still questions to be answered. Is the CSP header also
scrambled? Hopefully, this will be answered in an other article.

.. _AX5043: http://www.onsemi.com/pub/Collateral/AX5043-D.PDF
.. _programming manual: http://www.onsemi.com/pub/Collateral/AND9347-D.PDF
.. _code: https://github.com/medik/AX100-Tools/tree/master/Decapsulator
.. _main file: https://github.com/medik/AX100-Tools/blob/master/Decapsulator/main.m
