# Rogue Network Node

Linux AMD64 binaries for the RGN blockchain. Requires Ubuntu 22.04 or compatible,
Python 3, CA certificates and `libstdc++6`.

**FORGE V5 activates at block 1,121,800 — estimated 27 October 2026, 13:35 UTC.**

All RGN nodes, including exchange nodes, must upgrade before block **1,121,800**.

## Fresh installation

```sh
curl -fLO https://github.com/rogueisles/Rogue-Node/releases/download/v4.1.0/rgn-node-4.1.0-linux-amd64.tar.gz
curl -fLO https://github.com/rogueisles/Rogue-Node/releases/download/v4.1.0/SHA256SUMS
sha256sum --ignore-missing -c SHA256SUMS
tar -xzf rgn-node-4.1.0-linux-amd64.tar.gz
cd rgn-node-4.1.0-linux-amd64
sha256sum -c PACKAGE-SHA256SUMS
RGN_DATADIR="$HOME/.local/share/rgn" ./start.sh
```

The starter loads the bundled mainnet profile. RPC binds to localhost on port
18545; P2P uses TCP/UDP 30303.

## Upgrade an existing node

Complete the download, checksum and extraction steps above, stopping before the
start command. Reuse the existing data directory.

For a systemd-managed node, use your current service name and install the
extracted package at `/opt/rgn-node/4.1.0`:

```sh
sudo systemctl stop rgn-node.service
sudo mkdir -p /opt/rgn-node/4.1.0
sudo cp -a . /opt/rgn-node/4.1.0/
sudo systemctl edit rgn-node.service
```

Use these service overrides, replacing the data path with your existing path:

```ini
[Service]
ExecStart=
ExecStart=/opt/rgn-node/4.1.0/start.sh
Environment=RGN_DATADIR=/absolute/path/to/existing/rgn-data
```

```sh
sudo systemctl daemon-reload
sudo systemctl start rgn-node.service
sudo systemctl status rgn-node.service
```

Keep the existing service user and custom settings. Confirm peers and continuing
chain progress after startup; preserve the database and wallet files.

For a foreground installation, stop the old process with Ctrl+C, then run:

```sh
RGN_DATADIR="/absolute/path/to/existing/rgn-data" ./start.sh
```

## Configuration

| Variable | Default | Purpose |
| --- | --- | --- |
| `RGN_DATADIR` | `rgn-data` beside the starter | Persistent data directory; use an absolute path |
| `RGN_CACHE_MB` | `512` | Database cache |
| `RGN_MAX_PEERS` | `25` | Peer limit |
| `RGN_RPC_PORT` | `18545` | Local RPC port |
| `RGN_P2P_PORT` | `30303` | P2P port |

```sh
RGN_DATADIR="$HOME/.local/share/rgn" python3 health.py
```

Use the running node's actual data path for upgraded or service-managed nodes.
The health check reads its local IPC socket, independently of the RPC port.

## Changes in 4.1.0

Optional SOQ work joins DGB/DOGE, with registry support for compatible future
coins and treasury rotation. The target becomes 12.5 seconds at activation.
Genesis, balances and rewards per block remain unchanged.

- [Downloads](https://github.com/rogueisles/Rogue-Node/releases/tag/v4.1.0)
- [Complete source](https://github.com/rogueisles/Rogue-Network)
- [Protocol reference](https://github.com/rogueisles/Rogue-Network/blob/main/docs/FORGE-V5-REGISTRY.md)
- [Docker](https://github.com/rogueisles/Rogue-Docker-Node)
