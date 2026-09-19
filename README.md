# quantumtouch-addons-dist

The add-on catalogue for QuantumTouch BNG. One asset, `addons.json`, published
on every release and fetched by boxes from
`releases/latest/download/addons.json`.

## This is discovery, never authorisation

The catalogue says which versions *exist*. It does not grant permission to
install anything, and a box never trusts it for integrity: the `.sha256`
sidecar published beside each **artifact** is the trust anchor, and it is
verified before a single byte is extracted. A tampered catalogue can at worst
point a box at a URL whose checksum will not match — which fails closed.

This repo is **public on purpose**. Boxes fetch the manifest unauthenticated; a
private repo returns 404 to every customer and update checks silently do
nothing, which is exactly the failure this catalogue was created to end.

## Shape

```json
{
  "schema": 1,
  "addons": {
    "<module-id>": {
      "track": "latest" | "pinned",
      "versions": [
        { "version": "1.2.3", "url": "https://...", "sha256": "...",
          "notes": "...", "min_bng": "1.4.0" }
      ]
    }
  }
}
```

- **`track`** — `pinned` means a box must not be offered an upgrade even when a
  newer version is listed. Policy lives here as data so it can change without
  shipping code.
- **`min_bng`** — optional. A box below it sees the row greyed with a reason
  rather than hidden, because a missing row reads as a bug. **Fails open**: a
  box that cannot parse either version installs anyway, since refusing over an
  unreadable version number is worse than allowing one the catalogue merely
  suspects is too old. The artifact checksum still applies either way, so
  "allowed" never means "unverified".
- Order versions newest-first; boxes preserve catalogue order.

## Air-gapped boxes

An unreachable catalogue is a normal state, not an error. Boxes render "no
catalogue" and keep working, installing from a direct upload instead. Point
`addons_manifest_url` at an internal mirror, or set it to the empty string to
turn checks off entirely.

## Publishing

Add or edit the entry in `addons.json`, then cut a release with `addons.json`
and its `.sha256` as assets. Boxes read `releases/latest/download/addons.json`,
so the newest release is what the fleet sees.
