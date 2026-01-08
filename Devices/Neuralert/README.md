# Install Renode

* Download the [release version](https://github.com/renode/renode/releases/tag/v1.16.0) and add it to your `$PATH`.

# Compile the Firmware

* Follow the instructions in the [neuralert_firmware](https://github.com/IoMT-Lab/neuralert_firmware.git) repository.

* The output firmware includes **FBOOT** and **FRTOS** images located under `neuralert_firmware/img`:

    * For example:
      * `DA16200_FBOOT-GEN01-01-c7f4c6cc22_W25Q32JW.img`
      * `DA16200_FRTOS-GEN01-01-2aa77df370-006629.img`

# Run the Firmware on Renode

* The file `da16200.repl` specifies the hardware configuration of the board, including the CPU and peripherals such as UART.

* The file `da16200.resc` specifies how the firmware is loaded and executed. Please modify the `$fboot` and `$frtos` variables in this file to point to the correct image paths.

* Run the following command:
    ```
    renode da16200.resc
    ```
