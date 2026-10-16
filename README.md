# AXI4-Lite Master (SystemVerilog)

FSM-based AXI4-Lite master with a simple command/response user interface, plus a self-checking testbench.

## Features
- All 5 AXI4-Lite channels: AW, W, B, AR, R
- AW and W are driven together but tracked independently, so the slave can accept them in any order
- Simple user interface: `cmd_valid/cmd_ready` in, `rsp_valid` (1-cycle pulse) out
- Parameterized address and data width (`addr_w`, `data_w`)
- One outstanding transaction at a time

## FSM
```
s_idle -> s_wr_addr_data -> s_wr_resp -> s_idle     (write)
s_idle -> s_rd_addr      -> s_rd_data -> s_idle     (read)
```

## User interface
| Signal | Dir | Description |
|---|---|---|
| `cmd_valid` / `cmd_ready` | in / out | command handshake (ready only in `s_idle`) |
| `cmd_write` | in | 1 = write, 0 = read |
| `cmd_addr`, `cmd_wdata`, `cmd_wstrb` | in | address, write data, byte strobes |
| `rsp_valid` | out | 1-cycle pulse when the transaction completes |
| `rsp_rdata`, `rsp_resp` | out | read data, response (00 OKAY, 10 SLVERR, 11 DECERR) |

## Verification
Self-checking testbench with a simple behavioural slave (16 words):
1. Single write + read
2. Byte strobe (`WSTRB`) partial write
3. Back-to-back transactions (8 writes, then 8 reads)

AXI4-Lite has no real bursts, so "burst" here means back-to-back single transfers.

## Run
Requires [Verilator](https://www.veripool.org/verilator/) 5.x (and GTKWave for waveforms).
```
make sim     # build + run, generates axi4_lite_master.vcd
make wave    # open waveform
make clean
```
Expected output ends with `RESULT: PASS`.

## Limitations / next steps
- Slave in the testbench is always-ready (no random wait states)
- Error responses (SLVERR/DECERR) are not exercised yet
- Handshake rules are not yet checked with SVA assertions
- One outstanding transaction only (no pipelining)

## Structure
```
rtl/axi4_lite_master.sv
tb/tb_axi4_lite_master.sv
Makefile
```
