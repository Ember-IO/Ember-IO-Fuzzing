# Ember-IO

Ember-IO is an automated testing tool for embedded systems. More specifically, it's a fuzzing framework focused on monolithic firmware, supporting firmware built for ARM Cortex-M microcontrollers. We aim to assist fuzzing of firmware through alternate input generation and coverage feedback methods.

You can read the full paper regarding our work [here](https://arxiv.org/abs/2301.06689).

## Installation
``sudo apt install build-essential ninja-build pkg-config libglib2.0-dev libpixman-1-dev``

``git clone https://github.com/Ember-IO/Ember-IO-Fuzzing``

``cd Ember-IO-Fuzzing``

``export EMBER_BASE_DIR=$(pwd)``

``git submodule update --init --recursive``

``cd AFLplusplus``

``make``

``cd qemu_mode``

``./build_qemu_support.sh``

## Instructions

### Usage
Ember-IO is based on AFL++'s QEMU mode. In order to fuzz firmware with Ember-IO, the AFL++ interface is used.

To operate AFL++, a minimum of an input seed directory (using -i), and output directory (using -o) must be supplied to the program afl-fuzz, and the fuzzer must be set to QEMU mode (using -Q).

### Micro-controller Configuration
Several additional parameters are provided to QEMU to allow fuzzing of the target firmware
First, the embedded machine has to be selected with *-machine embedded_fuzz*
Second, the location of the file to used for MMIO input has to be specified. As this file is generated by AFL++, we use the substitute value @@ to inform AFL++ it needs to include this directory. Include the parameter *-fuzz_input @@*
Third, the location of the binary to test needs to be defined. This is simply a directory to a .ELF file. E.g. *~/test_firmware.elf*
Lastly, the parameters for the target device's memory map need to be configured. This includes both the SRAM and flash sections. The SRAM setup is performed with the *-sram_base* and *-sram_size* commands. For a device with 128KB of RAM at address 0x20000000, append *-sram_base 0x20000000 -sram_size 128k* to the command. Similarly flash, for a device with 1MB of flash at address 0x8000000, the parameters *-flash_base 0x8000000 -flash_size 1M* would be appended to the command.

All combined, this results in the command ``./afl-fuzz -i ./seeds -o ./output -Q ~/test_firmware.elf -machine embedded_fuzz -fuzz_input @@ -sram_base 0x20000000 -sram_size 128k -flash_base 0x8000000 -flash_size 1M``

Examples using Ember-IO on real world binaries can be viewed in our scripts [here](https://github.com/Ember-IO/Ember-IO-Experiments/tree/main/Fuzzing).

### Crash Analysis
After crashes have been identified by the fuzzer, they can be replayed for further analysis. This is done by running them in QEMU.
To debug a crash, run afl-qemu-trace with the same values for -sram_base, -sram_size, -flash_base and -flash_size. Load the ELF file with the -kernel parameter, and set the machine to embedded_fuzz. The MMIO input file is selected through the -fuzz_input parameter.
All combined, based on the previously given usage example, this results in a command similar to ``./afl-qemu-trace -sram_base 0x20000000 -sram_size 128k -flash_base 0x8000000 -flash_size 1M -kernel ~/test_firmware.elf -machine embedded_fuzz -fuzz_input ./output/crashes/1``
To assist in locating faults QEMU's debug outputs can be enabled. Useful ones include *exec*, *int* and *mmu*. To enable this extra output, add *-d exec,int,mmu* to the end of your afl-qemu-trace command. It's also possible to connect with GDB to this instance, by adding *-s -S* to the parameters, and using the *target remote* command in GDB.
NOTE: Monitoring in GDB can influence execution under certain circumstances. Be sure to verify the crash still occurs in the same manner as without GDB.

### Experiments
Information regarding our experiments and results will be made available [here.](https://github.com/Ember-IO/Ember-IO-Experiments)


## Cite Us
```
@inproceedings{emberio,
title = {{Ember-IO}: Effective Firmware Fuzzing with Model-Free Memory Mapped IO},
author = {Guy Farrelly and Michael Chesser and Damith C. Ranasinghe},
booktitle = {Proceedings of 18th ACM ASIA Conference on Computer and Communications Security (ASIA CCS)},
year = {2023}
}
```
