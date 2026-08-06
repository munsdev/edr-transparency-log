# edr-transparency-log

Daily anchor log for [Election Defense Reporting](https://report.1stand.org)'s hash-chained incident archive.

Each report submitted to the archive is SHA-256 hashed and chained to the one before it (`chain_hash = SHA256(content_hash + prev_chain_hash)`), so any retroactive edit to the archive breaks the chain from that point forward. That proves internal consistency, but on its own it doesn't prove *when* a given chain state existed, and it doesn't survive the archive itself being seized or destroyed.

This repo closes that gap. Once a day, a job on the archive's server:

1. Takes the current chain head (the `chain_hash` of the most recent report).
2. Gets an RFC 3161 timestamp token for it from [DFN's public timestamp authority](https://www.pki.dfn.de/) (Germany) — a trusted third party attesting "this exact hash existed at this exact time," independent of anything the archive operator controls.
3. Commits both the hash and the timestamp token here, to a public, forkable repo.

Because this repo is public and mirrored the moment anyone forks or clones it, no single actor — including whoever runs the archive — can quietly rewrite history without the discrepancy showing up against every existing copy.

**What this repo contains:** daily root hashes and timestamp tokens only. No incident data, no personal data, nothing that identifies anyone. It is safe to mirror and safe to publish.

Each day's record lives at `anchors/YYYY-MM-DD.json`:

```json
{
  "date": "2026-11-03",
  "root_chain_hash": "<sha256 hex of the latest report in the archive as of this run>",
  "rfc3161_token_base64": "<the DFN TSA's signed timestamp token, base64>",
  "tsa": "https://zeitstempel.dfn.de"
}
```

To verify a token independently: `openssl ts -reply -in token.tsr -text` (after base64-decoding `rfc3161_token_base64` back to binary).
