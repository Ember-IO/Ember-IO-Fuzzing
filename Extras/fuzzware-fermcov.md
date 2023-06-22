# FERMCov for Fuzzware


We include an implementation of our FERMCov technique for the [fuzzware emulator]([https://github.com/fuzzware-fuzzer/fuzzware-emulator). Our tests were performed using the version corresponding to commit ``c454cb9``.


To apply this implementation, apply the ``fuzzware-emulator.patch`` patch to Fuzzware's emulator directory.
Then move to the unicorn directory with ``cd unicorn/fuzzware-unicorn`` and apply the ``fuzzware-unicorn.patch`` patch file.
To generate the required header files, run ``cd qemu`` and execute ``./gen_all_header.sh``.

Rebuild Fuzzware's emulator using the ``install_local.sh`` script from Fuzzware's [main repository](https://github.com/fuzzware-fuzzer/fuzzware).
