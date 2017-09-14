.. title: Sofware Defined Radio with SEAM
.. slug: software-defined-radio-with-seam
.. date: 2017-09-11 10:02:51 UTC+02:00
.. tags: sdr
.. category: sdr
.. link: 
.. description: 
.. type: text

.. thumbnail:: /images/kth-sband-test.jpg
   :align: right
   
Okay, I am working currently working on a Software Defined Radio (SDR) solution
for the satellite SEAM which is going to measure the electromagnetic field
around the earth. The satellite can communicate with the ground station in two
ways: over the UHF channel (ham radio channel) or over S-band, the latter is
supposed to be the high bandwidth channel with a data rate of 3 Mbit/s. The main
purpose of the SDR project is to allowing flexibility in the signal processing
on ground station. Instead of having big and heavy electronic components doing
the signal processing we can just change the code inside the computer, thus
allowing us to recieve on both UHF and S-band over the same device.

The SDR device I am using is an Ettus B210 and the software that are doing all
the heavy lifting is Labview Communication 2.0, or at least *supposed* to do
that. The problem today is that the phase locking over S-band channel is not
working properly, and we can see it by examining the recovered signal onto a
phase diagram. Because of the sparse points in the diagram it is believed that
the reason for this is that phase locked loop are locking on the wrong
phase. That is because the demodulation module are averaging the samples,
including those from the wrong phase. How do you solve it? I don't know it
yet. So instead of working on the S-band channel I am currently focusing on to
getting it to work on UHF.





