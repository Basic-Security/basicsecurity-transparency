# basicsecurity.net transparency receipts

This repository holds what [basicsecurity.net](https://basicsecurity.net) **signs** about its corrections
ledger, with receipts from systems we don't operate. Our build pushes it automatically. Software Heritage
and the Internet Archive keep copies.

**What is here:** our own signed events, their hashes and receipts. **What is never here:** anything a
reader sent us. Corrections that readers submit stay in the ledger on our site
(<https://basicsecurity.net/corrections/ledger.jsonl>). Each head commits to that ledger by its
RFC 6962 Merkle root hash, so you can check the ledger against a head without the ledger being copied here.

| Path | What it is |
|---|---|
| `events/<id>.json` | A signed Nostr event, written once with the exact bytes we published: a transparency-log head (kind 1, tag `t=bsn-transparency-log`) or one of our errata (kind 1111, tag `t=bsn-erratum`; see below). |
| `ots/<id>.json.ots` | An [OpenTimestamps](https://opentimestamps.org) proof over the SHA-256 of that event file. |
| `rekor/<id>.sigstore.json` | A [Sigstore](https://www.sigstore.dev) bundle: our signature over the event file, logged in the public Rekor transparency log. |
| `transparency-signer.pub` | The ECDSA P-256 public key behind those Sigstore signatures. |
| `index.json` | A summary of all of the above. |

The Nostr key that signs the events (x-only, hex) is
`8d20e910e73e61784d3a871c4cfb4642848b94ce3034d59d9f7c2b7c9f32d311`
(`npub135swjy888eshsnf6suwye76xg2zgh9xwxq6dt8vl0s4he8ej6vgsyufrtw`). The site publishes it at
<https://basicsecurity.net/.well-known/bsn-corrections.json>.

## Verify, without trusting us

**1. The event is ours and unchanged.** The event's `id` is the SHA-256 of its NIP-01 serialization, and
`sig` is a BIP-340 Schnorr signature over that id by the key above. Any Nostr library checks both. Our own
stdlib verifier is `tools/nostr_crypto.py` in the site's source.

**2. It existed by a point in time (OpenTimestamps).**

```
ots verify -f events/<id>.json ots/<id>.json.ots
```

A new proof is *pending*: the calendars commit it to Bitcoin, usually within hours. Once a calendar has done
so, `ots upgrade ots/<id>.json.ots` fetches the Bitcoin attestation onto your local copy; our build does the
same each day and republishes the upgraded proof.

**3. It is in Sigstore's public log (Rekor).**

```
cosign verify-blob --key transparency-signer.pub --bundle rekor/<id>.sigstore.json events/<id>.json
```

The bundle carries the log entry's inclusion proof and a signed checkpoint, so the check works offline.
Rekor v2 has no search index, so these bundles are our record of where each entry lives.

**4. The ledger matches the head.** Fetch `ledger.jsonl` and recompute the RFC 6962 root
(`tools/verify_merkle.py` in the site's source). It must equal the event's `root` tag at the event's `size`.
A rewritten ledger will not match a head that is already signed, timestamped and logged here.

Licence: CC0 1.0. These are facts and receipts; use them freely.

## Errata

When we get one of our records wrong, we publish an erratum: a NIP-22 kind:1111 Nostr event rooted on the
record's URL and signed with the same build key as the log heads. It states what was wrong, what we changed,
when, and the cited source. Its `erratum-sha256` tag is the hash of the reviewed erratum file in our source,
and the site build refuses to ship if that file changes after publication. We never delete or replace an
erratum. A correction to one is a new erratum whose `supersedes` tag names the earlier event. The list is at
<https://basicsecurity.net/corrections/#errata>.
