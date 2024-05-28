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

## nusb

Embassy has several USB programs ready to go. For example, a fake HID mouse can be created on the RPi by:

```
cd embassy/examples
cargo run --release --bin usb_hid_mouse  <------- this will cause the mouse pointer to jump around the screen

lsusb  <------- this will show the connected HID device
```

For bulk HID USB transfers using ```nusb```:

```
cd embassy/examples
cargo run --release --bin usb_raw_bulk
```

Bulk transfers using ```nusb``` require some PC based code and access to the USB port:

```
sudo touch 70-plugdev-usb.rules
SUBSYSTEM=="usb", MODE="0660", GROUP="plugdev"  <------- copy this into the rules file
sudo nano 70-plugdev-usb.rules
sudo udevadm control --reload  <------- ensure new rules are used
sudo udevadm trigger <------- ensure new rules are applied to already added devices
```

To perform a USB transfer, ```main.rs``` is copied from within ```usb_raw_bulk.rs```:

```
use futures_lite::future::block_on;
use nusb::transfer::RequestBuffer;

const BULK_OUT_EP: u8 = 0x01;
const BULK_IN_EP: u8 = 0x81;

fn main() {
    let di = nusb::list_devices()
        .unwrap()
        .find(|d| d.vendor_id() == 0xc0de && d.product_id() == 0xcafe)
        .expect("no device found");
    let device = di.open().expect("error opening device");
    let interface = device.claim_interface(0).expect("error claiming interface");

    let result = block_on(interface.bulk_out(BULK_OUT_EP, b"hello world".into()));
    println!("{result:?}");
    let result = block_on(interface.bulk_in(BULK_IN_EP, RequestBuffer::new(64)));
    println!("{result:?}");
}
```

and ```cargo.toml``` is:

```
[package]
name = "nusb_rpi_embassy"
version = "0.1.0"
edition = "2021"

[dependencies]
nusb = "0.1.9"
futures-lite = "2.3.0"
```

```cargo run```:

```
Completion { data: ResponseBuffer { transferred: 11, .. }, status: Ok(()) }
Completion { data: [104, 101, 108, 108, 111, 32, 119, 111, 114, 108, 100], status: Ok(()) }
```

Nice !!!
