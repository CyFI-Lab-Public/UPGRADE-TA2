# Install Renode

* Download the [release version](https://github.com/renode/renode/releases/tag/v1.16.0) and add it to your `$PATH`.

# Compile the Firmware

* Follow the instructions in the [neuralert_firmware](https://github.com/IoMT-Lab/neuralert_firmware.git) repository.

* The output elf firmware locates under `build/`:

    * neuralert.elf

# Run the Firmware on Renode

* The file `neuralert.repl` specifies the memory mapping of the board.

* The file `neuralert.resc` specifies how the firmware is loaded and executed. Please modify the path in `sysbus LoadELF @/path/to/neuralert_firmware/build/neuralert.elf true`.

* Run the following command:
    ```
    renode neuralert.resc
    ```
