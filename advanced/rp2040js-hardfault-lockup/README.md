# Add ARMv6-M HardFault and lockup to the RP2040 core

| | |
|---|---|
| **Project** | [wokwi/rp2040js](https://github.com/wokwi/rp2040js) (MIT) |
| **Base commit** | [`a304c7486f`](https://github.com/wokwi/rp2040js/tree/a304c7486f329dae64be5f65baf28795d62ceb95) |
| **Language** | TypeScript |
| **Reference solution** | +174 / -34 lines across 2 files |
| **New tests** | +1,217 lines in 1 test file |

## Problem

I want us to implement the following faults.

Any instruction will fault if the instruction is a bkpt/udf and RP2040.onBreak has not been modified, if the instruction loads/stores from/to a region not listed below or if the instruction is being loaded from a region not listed as executable below. When an instruction faults the instruction itself will make no changes to registers/memory and the CPU will hardfault (exception 3) through its normal exception entry. The instruction address that faulted will be placed on the stack as the return address.

These regions include 0x00000000-0x00003FFF which is the boot rom, 0x10000000-0x1FFFFFFF is the xip region, 0x20000000-0x20041FFF is sram and its alias 0x21000000-0x2103FFFF. 0x40000000-0x5FFFFFFF contains all of the apb/ahb-lite peripherals. 0xD0000000-0xDFFFFFFF contains the sio block and finally 0xE0000000-0xE00FFFFF is the ppi. All of these may or may not have their contents simulated by the RP2040 emulator. The only regions that are considered executable are 0x00000000-0x00003FFF, 0x10000000-0x13FFFFFF and 0x20000000-0x20041FFF.

If we fault while Hardfault/NMI is active the cpu will lock up. If the cpu can't push the hardfault stack frame into a listed region it will lock up. When locked up reading pc will give you 0xFFFFFFFE and stepping will not do anything.

All memory operations done through DMA, gdb and using RP2040 read/write functions will operate as normal and not cause the CPU to fault.

## Files

- `test.patch`: the new tests, plus `test.sh`, which runs the existing suite (`base`) or the new tests (`new`)
- `solution.patch`: the reference solution (code, plus any build files and docs it updates; no tests)
- `Dockerfile`: the environment the tests run in
- `BASE_COMMIT.txt`: the upstream commit both patches apply to

## Run it

From this folder:

```bash
git clone https://github.com/wokwi/rp2040js repo && cd repo
git checkout a304c7486f329dae64be5f65baf28795d62ceb95
cp ../Dockerfile . && git apply ../test.patch
docker build -t rp2040js-hardfault-lockup .
docker run --rm rp2040js-hardfault-lockup ./test.sh --output_path /tmp/results.xml new   # fails: the change isn't made yet
git apply ../solution.patch && docker build -t rp2040js-hardfault-lockup .
docker run --rm rp2040js-hardfault-lockup ./test.sh --output_path /tmp/results.xml new   # passes
docker run --rm rp2040js-hardfault-lockup ./test.sh --output_path /tmp/results.xml base  # existing tests still pass
```
