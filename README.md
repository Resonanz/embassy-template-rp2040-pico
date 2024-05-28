This is a WIP.

## probe-rs

probe-rs does not work on a solitary RPi Pico board. Unlike the STM Nucleo boards that have built-in hardware for programming the microcontroller, the RPi Pico does not.

To use probe-rs using a second RPi Pico as a USB JTAG/debug probe:

* https://github.com/rp-rs/rp-hal
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

## memory.x

A file named ```memory.x``` with the following contents needs adding to the project folder root.

```
MEMORY {
    BOOT2 : ORIGIN = 0x10000000, LENGTH = 0x100
    FLASH : ORIGIN = 0x10000100, LENGTH = 2048K - 0x100

    /* Pick one of the two options for RAM layout     */

    /* OPTION A: Use all RAM banks as one big block   */
    /* Reasonable, unless you are doing something     */
    /* really particular with DMA or other concurrent */
    /* access that would benefit from striping        */
    RAM   : ORIGIN = 0x20000000, LENGTH = 264K

    /* OPTION B: Keep the unstriped sections separate */
    /* RAM: ORIGIN = 0x20000000, LENGTH = 256K        */
    /* SCRATCH_A: ORIGIN = 0x20040000, LENGTH = 4K    */
    /* SCRATCH_B: ORIGIN = 0x20041000, LENGTH = 4K    */
}
```
