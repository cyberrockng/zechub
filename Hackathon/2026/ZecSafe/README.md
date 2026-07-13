# ZecSafe — Lose one key, not your ZEC

**Track:** 👥 FROST — threshold signing and collaborative custody

**Source:** <https://github.com/cyberrockng/zecsafe>
**Live demo:** <https://zecsafe.vercel.app/demo>
**Demo video:** _(link in the PR description)_
**License:** MIT

A 2-of-3 FROST authorization control plane for shielded Zcash: one signer can
disappear and funds still move; one tampered byte in a transaction and nothing
signs.

## Verified Zcash mainnet interaction

The core requirement, first. ZecSafe executed a real 2-of-3 FROST
(re-randomized, Orchard/redpallas) signing ceremony **with one participant
unavailable** and broadcast the result to Zcash mainnet:

```text
Run ID:     p0-023-20260712T145358Z
Network:    main
Txid:       27d0e850202f3f2c37b7de0ded80bdaac1f9fef1fc663c7d6cf107fad55e8527
Status:     CONFIRMED (shielded), block 3,409,837
Bundle:     sha256:e4684eb1df7bbf48fda46ce4353968640f664c306b097e868e3b2ba780351b8d
```

Check it on any explorer:
<https://mainnet.zcashexplorer.app/transactions/27d0e850202f3f2c37b7de0ded80bdaac1f9fef1fc663c7d6cf107fad55e8527>

## What it does

- **Threshold custody that survives key loss.** A 2-of-3 FROST group signs
  shielded spends; the recorded mainnet run completed with signer C
  unavailable.
- **Binding Firewall.** Before any FROST round starts, the reviewed intent
  (recipient, amount, network, memo/fee policy, change) is compared field by
  field against the actual PCZT. A mismatch blocks signing — no signer is even
  contacted.
- **Human broadcast gate.** Software prepares, signs, proves, and combines the
  PCZT; releasing funds to mainnet requires explicit human approval.
- **Attack it yourself, in your browser.** The live demo page includes a
  Tamper Lab: the zecsafe-proof-v1 verifier reruns client-side (WebCrypto
  SHA-256 over the canonical bundle) against a local copy of the recorded
  proof — flip a txid character, swap a signer fingerprint, claim a 3-of-3
  threshold, or edit the JSON by hand, and watch the gates fail.
- **Tamper-evident public proof.** Every run emits a hash-bound
  `zecsafe-proof-v1` bundle with a one-command verifier and a tamper
  demonstration. Recipient, amount, and memo are withheld from public
  evidence; the commitment binds them without disclosing them.

## Run it / verify it yourself

```bash
git clone https://github.com/cyberrockng/zecsafe
cd zecsafe

# Re-verify every gate of the pipeline against the recorded run (hashes
# recomputed live; stops at the human broadcast-approval gate):
make proof-run-dry

# Verify the recorded mainnet run end to end:
make judge-proof-mainnet          # VERDICT: VERIFIED RECORDED ZECSAFE PROOF

# See a single edited byte get caught:
make judge-proof-mainnet-tamper   # VERDICT: TAMPER DETECTION PASS

# Local app (same evidence UI as the hosted demo):
npm start                         # http://127.0.0.1:4173/demo
```

Setup, architecture, threat model, and the full development/audit log are in
the source repo (`README.md`, `PROOF_SPEC.md`, `SECURITY.md`, `PRIVACY.md`,
`docs/`).

## Honest limitations

ZecSafe is a hackathon proof-of-concept built on re-randomized FROST tooling.
The referenced NCC audit of the ZF FROST repository did not cover re-randomized
FROST. ZecSafe is not audited production custody software, and its privacy
limits (coordinator visibility, signer linkability, network privacy) are
disclosed in the repo. The hosted demo page is a read-only evidence viewer by
design — it cannot spend funds.
