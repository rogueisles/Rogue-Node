# Rogue Network (RGN) — Linux Binary Node

Run a regular RGN full node without Docker or compiling from source. This
repository retains its historical `Ogg-Node` name; the current RGN software and
consensus source are maintained in [Rogue-Network](https://github.com/rogueisles/Rogue-Network).

**Upgrade before block 1,100,100.** Historical OGG/RGN rules and rewards remain
in effect through block **1,100,099**. The FORGE rules begin at **1,100,100**.
The original genesis, chain ID **70088**, accounts and contract state are
preserved. Existing OGG units remain one for one as RGN.

## Current download

Use the verified **RGN v3.0.0** files from the canonical
[release page](https://github.com/rogueisles/Rogue-Network/releases/tag/v3.0.0):

- [rgn-v3.0.0-linux-amd64.tar.gz](https://github.com/rogueisles/Rogue-Network/releases/download/v3.0.0/rgn-v3.0.0-linux-amd64.tar.gz)
- [SHA256SUMS](https://github.com/rogueisles/Rogue-Network/releases/download/v3.0.0/SHA256SUMS)

Older OGG releases in this repository are historical. Their binaries, genesis
configuration and wallet-only ProgPoW mining instructions do not provide this
FORGE upgrade. Do not use the old `releases/latest` download examples.

The package includes the `egem` node executable, the required shared mainnet
activation configuration, current peer configuration, `start-node.sh`, a
read-only `check-node.py` helper, setup instructions, licenses and checksums.

## Requirements

- Linux **x86-64 / AMD64**, with **glibc 2.34 or newer**. Ubuntu22.04 is the
  reference build/runtime environment; the released binary also passed a
  startup check on Ubuntu24.04. No Windows/macOS/ARM/musl binary is supplied.
- `curl`, `tar` and `sha256sum`; Python3 for the optional verification helper.
- SSD capacity for a growing full-chain database. The supplied startup profile
  uses512MiB cache; it is not a certified minimum total-RAM requirement.
- Native package ports: **8545 HTTP RPC on localhost**, **30666 TCP/UDP P2P**.

These native-package ports differ from the Docker examples, which use18545
and30303. Preserve custom ports when upgrading an existing deployment.

## 1. Download and verify

The following commands are for a **new node**. Existing operators should first
read the upgrade section below and stop their existing process cleanly.

```bash
mkdir -p "$HOME/rgn-download-v3.0.0"
cd "$HOME/rgn-download-v3.0.0"

curl --fail --location --remote-name https://github.com/rogueisles/Rogue-Network/releases/download/v3.0.0/rgn-v3.0.0-linux-amd64.tar.gz
curl --fail --location --remote-name https://github.com/rogueisles/Rogue-Network/releases/download/v3.0.0/SHA256SUMS

sha256sum -c SHA256SUMS
tar -xzf rgn-v3.0.0-linux-amd64.tar.gz
cd rgn-v3.0.0-linux-amd64
sha256sum -c PACKAGE-SHA256SUMS
```

## 2. Initialize a new node

Run as the account that will own the data. This example stores persistent data
in `$HOME/.rgn`.

```bash
./egem --datadir "$HOME/.rgn" --networkid 70088 --nousb init ./RGN-mainnet-1100100-genesis.json
```

Use the **bundled activation configuration**. The source repository's historical
`genesis.json` alone does not schedule the FORGE upgrade. Do not modify or
round the large integer values in the shared configuration.

## 3. Start a regular node

```bash
./start-node.sh "$HOME/.rgn"
```

This starts the node in the foreground, connects to the configured RGN peers,
and exposes HTTP RPC on **127.0.0.1:8545**. Press Ctrl+C for a clean shutdown.
A regular node needs **no WALLET, private key, DGB/DOGE daemon or signing service**.
It does not start mining.

To run it in the background instead, from the extracted release directory:

```bash
nohup ./start-node.sh "$HOME/.rgn" > "$HOME/.rgn/rgn-node.log" 2>&1 &
echo "$!" > "$HOME/.rgn/rgn-node.pid"
```

Use either the foreground or background method, not both. For permanent
operation across server reboots, put the same startup command under your
existing service manager with enough time for graceful database shutdown.

## 4. Verify identity, configuration and sync

In another terminal, from the extracted release directory:

```bash
python3 ./check-node.py "$HOME/.rgn"
```

The helper checks the exact original genesis, chain ID and full consensus
configuration and reports height, peers and sync state. It changes no node data.
Check that peers are connected and height catches up with the canonical network.
`eth_syncing=false` at height zero does not by itself mean synchronization is
complete.

Optional local HTTP checks:

```bash
curl -s http://127.0.0.1:8545 \
  -H 'Content-Type: application/json' \
  --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}'

curl -s http://127.0.0.1:8545 \
  -H 'Content-Type: application/json' \
  --data '{"jsonrpc":"2.0","method":"net_peerCount","params":[],"id":1}'

curl -s http://127.0.0.1:8545 \
  -H 'Content-Type: application/json' \
  --data '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}'
```

The startup profile uses `--syncmode full --gcmode full` for normal full-node
state garbage collection. This does not prune all historical block files or
make disk usage fixed. Archive-state requirements need a separately planned
configuration and storage budget.

## 5. Logs and clean shutdown

For the background example:

```bash
tail -f "$HOME/.rgn/rgn-node.log"
```

To stop it, from the extracted release directory, first match the recorded PID
against this executable:

```bash
rgn_pid=$(cat "$HOME/.rgn/rgn-node.pid")
test "$(readlink -f "/proc/$rgn_pid/exe")" = "$(readlink -f ./egem)" && kill -INT "$rgn_pid"
```

Wait for the process to exit before restarting or backing up its database.
If you use a service manager, use its normal stop/start commands instead.

## Existing-node upgrades

The four operator-managed RGN mainnet nodes are already upgraded/configured;
they do not need to repeat initialization for this documentation update.

For another existing OGG/RGN node:

1. Stop it cleanly and confirm its process exited.
2. Save a consistent backup of the stopped datadir, old executable and startup
   configuration. Preserve node/account keys.
3. From the extracted RGN release directory, apply the shared configuration to
   the **actual existing datadir**:

```bash
RGN_DATA="/absolute/path/to/your/existing/datadir"
./egem --datadir "$RGN_DATA" --networkid 70088 --nousb init ./RGN-mainnet-1100100-genesis.json
```

4. Update the existing service/start script to use the new executable, retaining
   its existing datadir, identity, ports and other deployment settings. Restart
   that service, then run:

```bash
python3 ./check-node.py "$RGN_DATA"
```

Stop on incompatible-genesis/configuration errors. Do not force a rewind,
reset genesis, delete chain data or substitute a testnet database. Do not run
two processes against the same datadir. `NODE-SETUP.txt` inside the archive
contains the full procedure.

## Mining and wallet addresses

The old OGG `export WALLET=...` example was for the old ProgPoW wrapper. It is
not a working FORGE solo-mining command, and this binary package's
`start-node.sh` does not enable solo mining from that environment variable.

FORGE mining needs compatible Scrypt-to-V4 mining software, an RGN work node,
DigiByte and/or Dogecoin native backend, and automatic approvals from the
approved **2-of-4 signing quorum**. Ordinary RGN validators remain RGN-only.

A complete wallet-configured solo deployment is separate and is not supplied
by this regular-node package. No private key or seed phrase is needed merely
to configure a public mining beneficiary address.

## Source and release identity

- Full consensus source: [Rogue-Network](https://github.com/rogueisles/Rogue-Network).
- Binary source commit: [`18367e0aea25899af7fe8d608f0a19fa420740a9`](https://github.com/rogueisles/Rogue-Network/commit/18367e0aea25899af7fe8d608f0a19fa420740a9).
- Binary SHA256: `896166f853530bf74e2282e9950097801e5d88b5d306a6a639ce05f7d243cf05`.
- Archive SHA256: `7700c7ef82ae695d5eab83c9ad4b0f39be8f449c28c1da9300b122e854dd2abf`.
- The inherited binary filename/banner remains `egem` / `1.0.3-syrius`. The
  source commit and checksum identify this release. The version banner's
  compiled network default does not override the explicit runtime network70088.

This repository hosts native-node documentation and optional download mirrors.
Its automatically generated source archives contain this repository's files;
use the linked **Rogue-Network** repository for the complete node source.

## Docker alternative

The same verified node binary is also published as
[`roguenetwork/rogue-network:3.0.0`](https://hub.docker.com/r/roguenetwork/rogue-network).
That image uses a persistent Docker volume and different default host ports;
follow its Docker instructions rather than mixing the two startup methods.
