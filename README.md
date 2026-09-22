# RX 6800 Mining Launchers (RVN / ETC / MEWC)

Batch launchers and extracted miner binaries for mining Ravencoin, Ethereum Classic and Meowcoin on
an AMD RX 6800, tuned so the card stays usable as a desktop GPU while mining. The logic worth
keeping is in the four `.bat` files - pool failover choices and the clock/power profiles with their
measured wattage and hashrate comments; the miners themselves are upstream releases.

**Suggested repo name:** `rx6800-miner-launchers`
**Stack:** Windows batch; TeamRedMiner 0.10.21, SRBMiner-Multi 3.6.1, meowcoin data; stratum pools (2Miners, Nanopool, rplant)
**Status:** active
**Last modified:** 2026-09-02

## What it does

- `start_rvn.bat` - TeamRedMiner `-a kawpow` against `rvn.2miners.com:6060` with
  `kawpow.eu-west.nanopool.org:10631` as failover. Desktop profile `--prog_config=B640` at
  1200 MHz core / 800 mV / 1600 MHz memory: about 120 W and 22 MH/s, leaving ~40% of the GPU free.
- `start_etc.bat` - TeamRedMiner `-a etchash` against `etc.2miners.com:1010` then
  `etc.f2pool.com:8118`. Profile `B320` at 1500 MHz / 850 mV / 1875 MHz (~110-130 W). Ships with
  `WALLET=YOUR_ETC_WALLET_ADDRESS` and an explicit guard that aborts until you edit it.
- `start_meowc.bat` - SRBMiner-Multi `--algorithm-gpu meowpow` with rplant EU and Asia endpoints,
  plain port 7120 plus TLS 17120 failovers, capped at 1400 MHz / 850 mV / 1800 MHz (~120-150 W).
  Comments record that the pool rejects bech32 `MEWC1...` addresses, so payouts go to the legacy
  `M...` address in the same wallet.
- `start_meowc_MAX.bat` - same coin, no clock caps: measured ~33.3 MH/s at ~210 W with a saturated
  GPU and a laggy desktop. Kept as the "while I am away" variant.
- `teamredminer/`, `srbminer/`, `meowcoin/` - extracted upstream releases with their
  `SHA256SUMS` / `expected.md5` verification files, plus `trm.zip` (75 MB).

## Layout

```
start_rvn.bat         KawPow via TeamRedMiner, wallet configured
start_etc.bat         Etchash via TeamRedMiner, wallet placeholder + guard
start_meowc.bat       MeowPow via SRBMiner, clock-capped profile
start_meowc_MAX.bat   MeowPow, uncapped profile
teamredminer/         teamredminer-v0.10.21-win (upstream)
srbminer/             SRBMiner-Multi-3-6-1 + zip + expected.md5 (upstream)
meowcoin/             meowcoin-win64.zip + data + SHA256SUMS (upstream)
trm.zip               TeamRedMiner archive, 75 MB
```

## Running it

Edit the `WALLET` line to your own address first, then double-click the `.bat` you want. Each one
`cd`s to `%~dp0<miner>` and runs the miner in the foreground, ending with `pause`.

## Notes

- Needs edits before publishing: `start_rvn.bat`, `start_meowc.bat` and `start_meowc_MAX.bat` all
  hardcode live payout wallet addresses (Ravencoin and Meowcoin legacy, plus a bech32 address in a
  comment). Replace them with placeholders like the ETC script already uses - an address is not a
  secret key, but it is your public financial identity and anyone watching the chain can label it.
- Do not commit the miner trees or `trm.zip`: they are third-party binaries (and are what antivirus
  flags), so publish only the `.bat` files and let users download their miners.
- The comments encode pool reachability as measured on 2026-08-31 - 2Miners and Nanopool RVN stratum
  endpoints had open TCP ports but no handshake at write time, which is why failover is configured
  the way it is.
- SRBMiner requires all failover pools comma-separated inside one `--pool` flag; repeating the flag
  errors with "You defined more pools than algorithms".
