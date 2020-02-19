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