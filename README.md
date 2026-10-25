# Rogue Network node 4.1.0

Required coordinated FORGE V5 upgrade at **block 1,121,800**. The estimate is
27 October 2026 around 13:35 UTC; height, not the clock, determines activation.
Every validating node, including exchange nodes, must upgrade before that block.

Linux AMD64; Ubuntu 22.04 or compatible, Python 3, CA certificates and libstdc++6.
Verify the release checksums, keep these files together, then start:

```sh
sha256sum -c PACKAGE-SHA256SUMS
chmod +x egem start.sh
./start.sh
```

For an existing node, stop it gracefully and back up its configuration and wallet
custody. Keep its existing data directory. Set RGN_DATADIR to that absolute path,
then use the new guarded starter. Do not delete or replace the chain database.
The bundled profile keeps the original genesis and historical rules and installs
the agreed future schedule. Upgrade safety gates/supervisors that pin old config.

RPC 18545 binds localhost; P2P 30303 uses TCP/UDP. WALLET selects a public work
beneficiary and does not start a solo miner. Keep RPC private. Run
`python3 health.py` for identity/profile checks, then also check connected peers,
syncing status and a current canonical height. Identity health alone is not proof
of synchronization.

The inherited `egem version` banner reports legacy client/default metadata.
Use the supplied starter: it explicitly selects chain 70088 and the approved RGN
genesis/profile. It refuses incompatible data rather than silently replacing it.

V4 remains in effect below 1,121,800. V5 permits eligible DGB/DOGE/SOQ subsets,
bounded future compatible coins and delayed 3-of-4 treasury administration. The
target becomes 12.5 seconds from activation. Rewards per block/history remain
unchanged. Ordinary validators do not need native DGB/DOGE/SOQ nodes.

Before H a coherent rollback can withdraw the future schedule. After V5 blocks
exist, retain V5 validation; reverting to V4 is unsafe. See the full protocol,
operator and qualification guides in the source release.

Download the Linux AMD64 archive from the GitHub Releases tab. Full source: https://github.com/rogueisles/Rogue-Network
Docker: https://hub.docker.com/r/roguenetwork/rogue-network

[Approved activation profile and generated review](config/MAINNET-V5-RELEASE.md).

The 4.1.0 packaging retains the tested node's internal build identifier `rgn400v5`. This identifier is opaque; the release version, scheduled profile and published checksums identify this package.
