# Linker scripts

I mentioned that ARM Cortex M devices expect the firmware to have a specific memory layout. I was
referring to the [vector table]. The contents of this vector table are used in the booting process
and in the interrupt mechanism. ARM Cortex-M devices expect this vector table to be located at
address `0x0000_0000`.

[vector table]: https://developer.arm.com/docs/dui0552/latest/the-cortex-m3-processor/exception-model/vector-table

The linker is ultimately in charge of the memory layout of a program but we can influence in the
process using a file named linker script.
