# Results

zexdoc / zexall (Frank Cringle) on the R800 of the ZEMMIX core, SiDi128 board
(2026-10-10, ZEXR800.BAT: `R800`, `ZEXDOC > ZEXDOC.TXT`, `ZEXALL > ZEXALL.TXT`):

| File | |
|---|---|
| `zexdoc_sidi128.txt` | zexdoc on the board: 63 OK, 4 R800 differences |
| `zexall_sidi128.txt` | zexall on the board: 35 OK, 32 R800 differences |
| `zexall_openmsx_r800.log` | zexall on openMSX with an R800 (log of the BDOS calls: every string twice) |

The board, openMSX and the Verilator model (`sim`, `make zexall`) give the same
CRC in every test of zexall (67 / 67), and the board and the model the same in
every test of zexdoc.  The zexdoc errors are the R800 behaviour: CPL / CCF and
the H flag (`<daa,cpl,scf,ccf>`), DD / FD before an opcode without IX / IY
(`ld <bcdexya>,<bcdexya>`) and SLL as SLA (`shf/rot`, two tests).
