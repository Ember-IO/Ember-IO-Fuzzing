# Triaging with GDB

GDB is a powerful tool to assit in debugging crashes, and provides a similar interface for embedded firmware to traditional desktop applications. However, there are some cases that need to be treated with care when using GDB with Ember-IO.


## Accessing Memory

One of the common actions performed during debugging is to probe the system's state to check for anomalies. However, memory accesses through GDB still follow the same validation requirements as the firmware's memory accesses. Dereferencing pointers in GDB (either through print, or from the function parameters that GDB sometimes prints on breakpoints) can result in the firmware crashing, if the memory is not mapped. Dereferencing pointers to MMIO will cause data to be read from the fuzzing input, which will change the outcome of future MMIO reads.


## Interrupt Timing

Tools such as breakpoints and stepping influence Ember-IO's perception of the number of blocks executed. This in turn changes the timing of interrupts in the program. If the change in timing is large enough that a different number of MMIO values are read prior to the interrupt being triggered, the values read from the fuzzer generated input for a given peripheral can change. As a workaround for this issue, we recommend placing breakpoints within interrupt handlers and executing up until the last interrupt before the state of interest. Breakpoints and single stepping can then be safely used in the timeframe before the next interrupt/program termination. This works as the number of blocks executed inside interrupt handlers are not counted towards the timer for the next interrupt.



For accurate usage in these cases, we recommend verifying the observations of GDB by executing the input with ``-d exec,cpu``. This allows you to view the contents of the CPU registers at each block during execution, until the crash.
