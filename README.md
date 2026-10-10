# R800

The R800 CPU of the MSX turbo R for FPGA, as used by the
[ZEMMIX](https://github.com/BigMist/ZEMMIX) core.

The CPU is derived from **NextZ80** by Nicolae Dumitrache (OpenCores, 2011,
LGPL 2.1; his header is kept in every file). NextZ80 does one synchronous bus
access per clock and about one clock per opcode byte, which is the R800
timing model, so it only needs to be held to run at the exact R800 speed.

## Files

| File | |
|---|---|
| `rtl/r800_cpu.v`, `rtl/r800_alu.v`, `rtl/r800_reg.v` | the CPU, module `R800` |
| `rtl/r800_bus.vhd` | entity `r800_bus`: the CPU on a T80s like bus (`CEN`, `WAIT_n`, `M1_n`, `MREQ_n`, `IORQ_n`, `RD_n`, `WR_n`, `RFSH_n`) |
| `rtl/r800_timing.vhd` | entity `r800_timing`: holds the CPU to the R800 time (openMSX rules) |
| `sim/` | Verilator models and tests |

## R800 behaviour

From openMSX (`CPUCore.cc`, `IS_R800`) and the real R800:

- `MULUB A,r` (ED C1/C9/D1/D9): HL = A x r; `MULUW HL,rr` (ED C3/F3):
  DE:HL = HL x rr. Flags: S = V = 0, Z = product zero, C = product does not
  fit in 8 / 16 bits, H, N kept.
- `SLL` (CB 30-37) is `SLA`; DD / FD CB d 30-37 only set the flags (C = A
  bit 7, the others 0).
- DD / FD before an opcode that does not use IX / IY is a NOP of two bytes
  (DD DD and DD ED too).
- X / Y flags are never taken from the result; `CPL` keeps the flags and sets
  H and N; `CCF` keeps H; `BIT` keeps S, V, C.
- Block repeats (LDIR, CPIR, INIR, OTIR...) fetch the instruction again on
  every iteration; OUTI / OTIR put B - 1 on the bus, as the Z80.

Not done: `LD A,I` / `LD A,R` do not hold the interrupts on the R800.

## Bus and timing

`r800_bus` turns every access of the CPU into a T1 / T2 / T3 bus cycle on
`CLK` (21.48 MHz on the MSX): T2 lasts until `WAIT_n` is high, the data is
taken at the end of T3, so an access takes at least 3 clocks, one R800 cycle.
`CEN` low freezes the CPU; the reset is taken even while frozen. `RFSH_n` is
not an R800 refresh cycle but a request to refresh the SDRAM of the host
(low 4 clocks every 128).

`r800_timing` counts the time a real R800 would take (1 cycle per opcode byte
or access, DRAM page breaks, ROM and slot waits, I/O 6 cycles on an even
cycle, VDP accesses 62 cycles apart, refresh 25 cycles every 210, MULUB 14 /
MULUW 36 cycles) and stalls the CPU with `CEN` at the T1 of an M1 while it is
early.

On the ZEMMIX, R800BEN gives (system timer ticks, measured / openMSX): NOP
2874 / 2828, LD A,(HL) 1521 / 1513, PUSH / POP 1CF9 / 1CDE, CALL / RET 0FEF /
0FE0, LDIR 247F / 2476, MULUB / W 2A28 / 2A1B, IN A,(n) 1A58 / 1A4B: all
within 0.7 %.

## Tests

Verilator and pasmo:

    cd sim
    make test

`tb.cpp` runs a CP/M program (`cpm.asm` at 0000h, the program at 0100h) with
the memory read in the same clock; `tbw.cpp` holds every access with random
waits, `tbi.cpp` adds a periodic IM 1 interrupt (cleared by `OUT (3)`),
`tbio.cpp` logs the I/O accesses. Expected:

    multest   03A8 0000 01 / 0000 000A 40 / 00E1 000F 00 / 0060 0626 01 / 2000 0000 00 / 0200 000A 01
    pfxtest   0500 0000 00 / 1234 000A 00 / F000 000A 84 / 5678 000A 84 / 1100 000A 29 / 1100 000A 00
    iotest    OTIR!0
    inttest   0653 8000 8000 0000 0000 (1619 INT)
    halttest  0800 8000 8000 0000 0000 (2049 INT)

zexdoc / zexall (put `ZEXDOC.COM` / `ZEXALL.COM` of Frank Cringle in `sim/`;
hours): `make zexdoc`, `make zexall`. The zexall CRCs of the R800 differ from
the Z80 ones; `zexcmp.py openmsx.log zexall.log` compares them with a run of
openMSX with an R800 (a log of the BDOS calls).

`results/` has the zexdoc / zexall output of the core on a SiDi128 board: the
same CRCs as openMSX with an R800 and as the model.

Simulation notes: with an event driven simulator (nvc, iverilog) the registers
are initialised in `initial` blocks, not in their declaration.

## License

LGPL 2.1, as NextZ80 (see `LICENSE`).
