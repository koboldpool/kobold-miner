# Kobold Miner

Kobold Pool's own miner for **BOUND** and **NEB**, on NVIDIA and AMD graphics cards (and the CPU for BOUND).
**No dev fee.** It mines only on [Kobold Pool](https://koboldpool.com): that is how the pool pays for it.

Download it from the [releases](../../releases) page (Windows and Linux). Only the normal graphics driver is needed.

## Quick start

```
kobold-miner --list                                   # the GPUs it finds
kobold-miner -u YOUR_BOUND_ADDRESS.rig1 -g 0          # BOUND on NVIDIA card 0
kobold-miner -u YOUR_BOUND_ADDRESS.rig1 -g 0 --opencl # BOUND on AMD card 0
kobold-miner -u YOUR_BOUND_ADDRESS.rig1 -t 8          # BOUND on 8 CPU threads
kobold-miner -u YOUR_NEB_ADDRESS.rig1 -g 0,1          # NEB on cards 0 and 1
```

The coin follows from the address (`B...` mines BOUND, `neb1...` mines NEB). `.rig1` is the worker name shown on the
pool's dashboard.

## Options

| Option | Meaning |
|---|---|
| `-u ADDRESS[.worker]` | your payout address and a worker name |
| `-g 0,1` | the GPUs to mine on (numbers from `--list`) |
| `--opencl` | BOUND: use OpenCL instead of CUDA (needed for AMD cards) |
| `-t N` | BOUND: CPU threads (default: all, or none when GPUs are given) |
| `-p solo` | mine solo instead of PPLNS |
| `-o host:port` | pool server (default: `bound.koboldpool.com:3480`, `neb.koboldpool.com:3460`) |
| `--dataset FILE` | NEB: the dataset file (default: `neb-datasets.bin` next to the miner, included in the download) |
| `--bench S` | BOUND: test each card for S seconds without a pool |
| `--pl W` | NVIDIA: power limit in watts |
| `--cclk MHz`, `--mclk MHz` | NVIDIA: lock the core / memory clock (`0` removes the lock) |
| `--coff MHz`, `--moff MHz` | NVIDIA: core / memory clock offset |

`kobold-miner --help` shows all options.

## Status line

Every 30 seconds the miner prints each card's speed, temperature, power and fan speed, and the accepted shares:

```
292214 H/s | GPU0 RX 9070 XT: 177.0 kH/s 58C 234W fan 35% | GPU1 RTX 4060: 115.2 kH/s 63C 90W fan 45%, shares accepted 40 rejected 0
```

`-` means the card doesn't report that value.

## NVIDIA clocks and power limit

`--pl`, `--cclk`, `--mclk`, `--coff` and `--moff` work as in OneZeroMiner.

- One value sets every card; a list sets each card in `-g` order (`--pl 90,125`).
- Run the miner as administrator (Windows) or with `sudo` (Linux), or the settings are skipped.
- The settings stay after the miner stops. Undo them with `--cclk 0 --mclk 0 --coff 0 --moff 0`; the power limit
  resets when the PC restarts.
- For AMD cards, use AMD's own software (Adrenalin).

## Speeds

Measured on our own cards:

| Card | BOUND | NEB |
|---|---|---|
| RX 9070 XT | 175 kH/s | 8.5 G/s screened |
| RTX 4060 (90 W) | 115 kH/s | 3.7 G/s screened |
| RTX 2060 | 100 kH/s | - |

BOUND speed depends mostly on memory speed: a memory clock locked low (by another miner) slows it down a lot.

## Safety

- Every share a GPU finds is checked on the CPU before it is sent.
- The miner needs only your address: no wallet file, no private key.
- Check the download against `SHA256SUMS` (`sha256sum -c SHA256SUMS` on Linux,
  `certutil -hashfile kobold-miner.exe SHA256` on Windows).

## Why only Kobold Pool

Kobold Pool signs every job it sends, and the miner checks the signature, so it doesn't work on other pools. The pool
software is open source: [bound-pool](https://github.com/koboldpool/bound-pool) and
[neb-pool](https://github.com/koboldpool/neb-pool).

## License

Free to use for mining on Kobold Pool; see `LICENSE`. Open-source parts and their licenses: `NOTICE`.
