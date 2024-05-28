This is a WIP.

## probe-rs

probe-rs does not work on a solitary RPi Pico board. Unlike the STM Nucleo boards that have built-in hardware for programming the microcontroller, the RPi Pico does not.

(To use probe-rs, see this page for using a second RPi Pico as a USB JTAG/debug probe: https://github.com/rp-rs/rp-hal.)

To use elf2uf2, you must first install ```elf2uf2-rs```

```
cargo install elf2uf2-rs --locked
```

The usual ```.cargo/config.toml``` file runner (e.g. ```runner = "probe-rs run --chip STM32L010RBTx"```) then needs replacing with ```elf2uf2-rs```.

```
[target.'cfg(all(target_arch = "arm", target_os = "none"))']
runner = "elf2uf2-rs -d"

[build]
target = "thumbv6m-none-eabi"        # Cortex-M0 and Cortex-M0+

[env]
DEFMT_LOG = "debug"
```
