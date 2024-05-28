This is a WIP.

## probe-rs

probe-rs does not work on a solitary RPi Pico board. Unlike the STM Nucleo boards that have built-in hardware for programming the microcontroller, the RPi Pico does not.

To use probe-rs using a second RPi Pico as a USB JTAG/debug probe:
* https://github.com/rp-rs/rp-hal.)
* https://reltech.substack.com/p/getting-started-with-rust-on-a-raspberry

To use elf2uf2, you must first install ```elf2uf2-rs```

```
cargo install elf2uf2-rs --locked
```

The Runner in ```.cargo/config.toml``` needs replacing with ```elf2uf2-rs```.

```
[target.'cfg(all(target_arch = "arm", target_os = "none"))']
runner = "elf2uf2-rs -d"

[build]
target = "thumbv6m-none-eabi"        # Cortex-M0 and Cortex-M0+

[env]
DEFMT_LOG = "debug"
```
