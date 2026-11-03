# Zcash Agora: the public log

This repository is the mirror of the [Zcash Agora](https://github.com/nikkolasg/zcash-agora)
ledger: the complete, append-only record of its polls. The ingest server
pushes here after every checkpoint. Browsers, auditors and authorities
read from here; nobody needs to contact the server to verify anything.

Vote and read the results at <https://nikkolasg.github.io/zcash-agora/>.

## What is in here

| Path | Content |
|---|---|
| `checkpoint.txt` | the latest signed checkpoint: the log's head, index and time, under the log key |
| `checkpoints/<n>.txt` | every checkpoint, in order |
| `entries/<n>.json` | the entries of checkpoint `n`: each registration, ballot and tally as received, with its proof and the ingest's receipt |
| `setup/ledger.json` | the ledger setup: origin, log key, ingest key, relays |
| `registries/<id>.json` | each registry: who may register (the ZKPassport policy) |
| `polls/<id>.json` | each poll: question, options, opening and closing times, drand round, circuit hash |
| `stamps/<n>.ots` | OpenTimestamps proofs anchoring checkpoints in Bitcoin |
| `circuits/<name>/` | the compiled Noir circuit and its verification key; its hash is pinned by each poll |

The exact layout is isoloir's: see [isoloir](https://github.com/lanterne-labs/isoloir),
`SPEC.md`, "Mirror".

## What you can check from here alone

- **That a ballot is in the log**: find it under `entries/` by its hash
  (the receipt on the vote page shows it and the checkpoint number) and
  check that checkpoint carries the log key's signature.
- **That every ballot came from one registered person**: each ballot
  carries a proof against the registry; `isoloir audit` replays all of
  them with circuits you built yourself.
- **That nothing was read early**: every ballot is time-locked to a
  [drand](https://drand.love) round that the poll names; the beacon of that
  round is public and dated.
- **That the tally is right**: open every ballot with that beacon and
  count. Two correct tallies are the same tally.

```sh
git clone --recursive https://github.com/nikkolasg/zcash-agora.git
cd zcash-agora && scripts/audit.sh        # reads this mirror, the relays and drand; prints PROBLEM lines on anything it cannot verify
```

## Why git

Git history makes every push a dated, hash-chained event: a checkpoint
cannot be replaced quietly, and jsDelivr serves each commit by hash, so
the vote page pins exactly what it read. Forks of this repository are
welcome; a fork that stays identical is an audit.

Everything here is public by design. Ballots are pseudonymous: the log
links a ballot to a registration, never to a passport or a name.
