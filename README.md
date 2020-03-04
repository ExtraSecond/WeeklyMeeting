# Mar 4

## Multi-threading kernel
- Using different core to do multi threading(Thus saved tons of code for context switch stuff...)
- Lock to stop data race

## GPIO application
- In one thread we implemented a GPIO application that lights up an LED.

# Feb 26

## UART and GPIO driver
- Migrated to RPi 3 now
  - `uart` to `usb tty`
- Define `driver` interface for device driver
  - Define `Result<(), ()>` for async task
  - Define `DeviceDriver` for device compatibility and device initialization
- Define `console` interface for input/output
  - Define `Read`, `Write`, `Statistics` trait
  - Impl `console` on the `uart` device
- Implemented `Mutex` as the lock for future possible multi-process
  - Got rid of unsafe raw pointer
- `UART` and `GPIO` driver
  - Write a wrapper of `GPIO` and `UART` register according to the instruction of Broadcom 2835
  - Implement the basic `read` and `write` for `GPIO` pins
  - Impl `console` interface for `UART`

# Feb 19

## Stop using L4ka
- No maintenance
- Hardly any documentation

## Writing an OS from scratch
- Based on QEMU `qemu-system-aarch64` now
  - Migrate to RPI3 in one day.
- Free-standing binary done.
  - No std library, only library available is `core`
  - No main, `kernel_init` will run forever and don't return.
- Hello World in kernel
  - Impl `core::fmt::Write` on a dummy struct
  - Rewrite `println!`, `print!` and `panic!` marco.
  - Cons: Every time we print, the dummy struct  will have to be created.
- Zero-cost abstraction
  - Got rid of asm as `cortex-a` crate provides all zero cost abstraction over asm.
- `uart` driver: under construction
  - Once done, we can migrate to RPI3

## Creating a memory management model
- Address translation 
  - 4 KiB page descriptor.
  - 40 bit memory addressing
  - 4 levels in the table hierarchy.
- Virtual memory: map low address (0x8 0000) to high address (0xFFFF FFFF 0008 0000)
  - Load kernel to high address
  - Load user code to low address

- Expected memory model:
```
       4G -----------> +----------------------------+------------0x FFFF FF01 0000 0000
                       |                            |
                       |            ...             |
                       |                            |
   TIMESTACKTOP -----> |----------------------------+------------0x FFFF FF00 0175 1000
                       |          TIMESTACK         |
                       |----------------------------+------------0x FFFF FF00 0175 0000
                       |            ENVS            |
                       |----------------------------+------------0x FFFF FF00 0170 0000
                       |            PAGES           |
                       |----------------------------+------------0x FFFF FF00 0140 0000
                       |       Kernel Page Table    |
      KSTACKTOP -----> +----------------------------+------------0x FFFF FF00 0100 0000
                       |         Kernel stack       |
                       |----------------------------|
                       |         Kernel Text        |
       KERNBASE -----> +----------------------------+------------0x FFFF FF00    8 0000
                       |          reserved          |
                       +----------------------------+------------0x FFFF FF00      0000




       4G -----------> +----------------------------+------------0x      0001 0000 0000
                       |                            |
                       |            ...             |
                       |                            |
      USTACKTOP -----> +----------------------------+------------0x           8000 0000
                       |         User stack         |
                       |----------------------------|
                       |         User Text          |
       0 ------------> +----------------------------+------------0x                0000
```