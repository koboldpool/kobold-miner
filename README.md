# Kobold Miner

Kobold Pool's own miner for **BOUND** and **NEB**, on NVIDIA and AMD graphics cards (and the CPU for BOUND). It is free:
**no dev fee**. It mines only on [Kobold Pool](https://koboldpool.com): that is how the pool pays for it.

Downloads are on the [releases](../../releases) page, for Windows and Linux, with `SHA256SUMS`. Only the normal graphics
driver is needed (no CUDA Toolkit or other SDK).

## Quick start

```
kobold-miner --list                                   # the GPUs it finds, CUDA and OpenCL numbering
kobold-miner -u YOUR_BOUND_ADDRESS.rig1 -g 0          # BOUND on NVIDIA card 0 (CUDA)
kobold-miner -u YOUR_BOUND_ADDRESS.rig1 -g 0 --opencl # BOUND on OpenCL card 0 (AMD)
kobold-miner -u YOUR_BOUND_ADDRESS.rig1 -t 8          # BOUND on 8 CPU threads
kobold-miner -u YOUR_NEB_ADDRESS.rig1 -g 0,1          # NEB on OpenCL cards 0 and 1 (NVIDIA and AMD)
kobold-miner -u YOUR_BOUND_ADDRESS.rig1 -g 0 --pl 90 --cclk 2500   # NVIDIA card 0 at 90 W, core locked at 2500 MHz
```

The coin follows from the address: a `neb1...` address mines NEB, a `B...` address mines BOUND. `.rig1` is the worker
name shown on the pool's dashboard.

## Options

| Option | Meaning |
|---|---|
| `-u ADDRESS[.worker]` | your payout address, and a worker name |
| `-g 0,1` | the GPUs to mine on: CUDA numbering for BOUND, OpenCL numbering with `--opencl` and for NEB (see `--list`) |
| `--opencl` | BOUND: use OpenCL instead of CUDA (AMD cards; works on NVIDIA too) |
| `-t N` | BOUND: CPU threads (default: all, or none when GPUs are given) |
| `-o host:port` | the Kobold Pool server (default `bound.koboldpool.com:3480`, `neb.koboldpool.com:3460`; solo: 3490 / 3470) |
| `-p password` | `solo` mines solo on the normal port |
| `--dataset FILE` | NEB: the network's dataset file (default `neb-datasets.bin` next to the miner; it is in the download) |
| `--batch N`, `--lanes 4\|1` | BOUND: hashes per GPU launch (default: tuned at start) and threads per hash |
| `--bench S` | BOUND: check each card against the CPU, then measure it for S seconds (no pool) |
| `--list` | list the GPUs |
| `--pl W` | NVIDIA: power limit in watts |
| `--cclk MHz`, `--mclk MHz` | NVIDIA: lock the core / memory clock (`0` clears the lock) |
| `--coff MHz`, `--moff MHz` | NVIDIA: core / memory clock offset (may be negative) |

`kobold-miner --help` prints the same list.

## Status line

Every 30 seconds the miner prints one line: the speed of each GPU, its temperature, power draw and fan speed, and the
shares the pool accepted and rejected.

```
1795343615 187491 H/s | GPU0 RX 9070 XT: 177.0 kH/s 58C 234W fan 35% | GPU1 RTX 4060: 115.2 kH/s 63C 90W fan 45%, shares accepted 40 rejected 0
1795343395 | GPU0 RX 9070 XT: 22.45 MA/s (8.41 G/s screened) 58C 230W fan 42% | GPU1 RTX 4060: 9.92 MA/s (3.72 G/s screened) 63C 87W fan 34%, shares accepted 1 rejected 0
```

The first line is BOUND (the first number is the total of all GPUs and CPU threads), the second NEB. `GPU0`, `GPU1`
follow the order of `-g`. The readings come from the graphics driver that is already installed: NVML on NVIDIA cards
(Windows and Linux), AMD's driver library on Windows, and the kernel's sensors (`/sys/class/hwmon`) for AMD on Linux.

- `-` means the card does not report that value (an idle fan reads `0%` on cards that stop their fans).
- `n/a` means nothing reports on that card at all.

Reading the sensors never stops or slows mining.

## NVIDIA clocks and power limit

The same options as OneZeroMiner (`--pl`, `--cclk`, `--mclk`, `--coff`, `--moff`), so settings from another miner can be
copied over. They are set when each card starts mining.

- **One value or one per card:** `--pl 90` sets every card in `-g`. `-g 0,1 --pl 90,125` gives card 0 90 W and card 1
  125 W. Leave an entry empty to leave that card alone: `-g 0,1 --coff ,150` changes only card 1.
- **Administrator rights are needed.** On Windows, open the Command Prompt with *Run as administrator* (or right-click
  your start file and choose *Run as administrator*). On Linux, run the miner with `sudo`. Without them the miner prints
  `needs administrator rights` and mines with the card's current settings.
- **Locks and offsets:** `--cclk` and `--mclk` hold the clock at that value; `--coff` and `--moff` move the card's
  whole clock curve up or down. Memory offsets are counted the way HiveOS and Linux tools count them (`--moff -2000`
  on an RTX 2060 is HiveOS's memory -2000).
- **They stay after the miner stops**, as with other miners. To undo them: `--cclk 0 --mclk 0` clears the locks
  (handy when another miner left the memory clock locked low, which slows BOUND), `--coff 0 --moff 0` clears the
  offsets, and `--pl` with the card's default wattage (or a restart of the computer) restores the power limit.
  `nvidia-smi -rgc` and `nvidia-smi -rmc` (as administrator) also clear the locks.
- Not every card can do every setting (an RTX 2060 cannot lock its memory clock). The miner prints what it set and
  what failed, for example:
  ```
  GPU 0: power limit 90 W, core offset +250 MHz, core clock locked at 2500 MHz, memory clock locked at 810 MHz
  ```
- AMD cards: set clocks and power in AMD's own software (Adrenalin on Windows); the options skip AMD cards.

## Speeds

Measured with this miner on our own cards:

| Card | BOUND | NEB |
|---|---|---|
| RX 9070 XT | 175 kH/s | 32-64 MA/s (8.5 G/s screened) |
| RTX 4060 (90 W) | 115 kH/s | 15-29 MA/s (3.7 G/s screened) |
| RTX 2060 (stock memory clock) | 100 kH/s | - |

BoundHash needs 512 KiB per hash and moves about 2 MiB per hash, so memory speed decides the BOUND rate. NEB's rate in
attempts per second depends on the current models of the chain (the share of training samples a model already handles),
so it moves from block to block.

## Safety

- Every share a GPU finds is computed again on the CPU with the coin's reference code before it is sent; a wrong result
  is counted as a GPU error and never sent. Each card also checks itself against the CPU when it starts.
- The miner talks only to the pool server you give it. It needs no wallet file and no private key: only your address.
- Check a download before you run it: `sha256sum -c SHA256SUMS` (Linux) or `certutil -hashfile kobold-miner.exe SHA256`
  (Windows) and compare with `SHA256SUMS`.

## Why only Kobold Pool

Each job the pool sends is signed by Kobold Pool for that connection, and the miner checks the signature. Other pools
cannot produce it, so the miner does not work there. The pool software and its payouts are open source:
[bound-pool](https://github.com/koboldpool/bound-pool) and [neb-pool](https://github.com/koboldpool/neb-pool).

## License

Free to use for mining on Kobold Pool; see `LICENSE`. Open-source parts and their licenses: `NOTICE`.
