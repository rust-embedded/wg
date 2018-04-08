# Exceptions

Our program now boots but it doesn't support exceptions as these are missing from the vector table.
Right now if the program encounters an exception condition at runtime it will try to use some
program instruction as an exception vector and this will most likely cause the program to crash and
raise a HardFault exception, which will repeat the process.

Unlike what we have done so far, which kind of has been done behind the user back, exceptions need
to be exposed to the user: the user must be able to choose the handler for each exception.

There are a few approaches to achieve this; we'll document all of them.

NOTE For simplicity in this guide we'll only deal with the set of exceptions common to all ARMv
Cortex-M3 devices and not with the device specific exceptions (interrupts).
