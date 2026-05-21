# OpenScientist Plugins — Verified Index

This repository is the **canonical index** of plugins that OpenScientist users
can install via `plane-tool plugins install <name>`. Inclusion in this index
is the "verified" tier (Track 2). Anyone can publish plugins to their own
ghcr.io namespace and install them directly without being listed here — see
[the unverified track](#unverified-track) below.

The index is two JSON files:

- **[`index.json`](./index.json)** — the live list of verified plugins.
- **[`yanked.json`](./yanked.json)** — recalled versions (CVEs, broken
  releases, withdrawals). Plane-tool checks this at install and at update.

Both files conform to the JSON Schemas under [`schema/`](./schema/).

---

## How verification works (the trust chain)

```
   plane-tool plugins install <name>
              │
              ▼
   1. Resolve <name> in index.json  ──►  { ref, trusted_identity, ... }
              │
              ▼
   2. Pull OCI manifest from <ref>
              │
              ▼
   3. cosign verify --certificate-identity-regexp '^<trusted_identity>@refs/(tags|heads)/.+$' \
                    --certificate-oidc-issuer https://token.actions.githubusercontent.com
              │
              ▼
   4. Pull layers, verify sha256 digests against manifest
              │
              ▼
   5. Atomic extract to ~/.openscientist/plugins/<id>/
```

Every plugin in this index has:

1. A public OCI artifact at `ghcr.io/<publisher>/<name>` (or any OCI registry).
2. A Sigstore signature anchored to a specific GitHub Actions workflow
   identity (the `trusted_identity` field).
3. A monotonically-tagged release history (`vMAJOR.MINOR.PATCH`).

A signature from the **wrong workflow** (e.g., a fork, a different branch,
or a malicious re-tag) fails verification client-side. The OCI registry is
content-addressable — once a tag is published, its sha256 digest is
immutable; plane-tool pins the digest after first install so subsequent
updates can detect tag-replays.

---

## Submitting a plugin

You publish to your own ghcr.io namespace first, then open a PR here to add
your entry. See [SUBMITTING.md](./SUBMITTING.md) for the full procedure.

Minimum checklist:

- [ ] Your plugin source repo is public on GitHub.
- [ ] `.github/workflows/release.yml` matches the [reference template](./templates/release.yml).
- [ ] Tag a release (`git tag v0.1.0 && git push --tags`) — verify the
      Action published an artifact at `ghcr.io/<you>/<plugin>:0.1.0` *and*
      that `cosign verify` succeeds against it.
- [ ] Your `plugin.json` follows the [OpenScientist plugin spec](https://github.com/fuzzy-dynamics/gitclines/blob/master/plugin_architecture.md).
- [ ] A clear README, an icon (~128×128 PNG), reasonable capability list.
- [ ] Open a PR adding your entry to `index.json`. The CI lints the JSON
      schema; humans review the rest.

---

## Unverified track

You don't need to be in this index to ship a plugin. Users can install any
signed plugin from any public OCI registry directly:

```bash
plane-tool plugins install ghcr.io/alice/witsoc-extras@1.0.0 --allow-untrusted
```

Plane-tool still verifies the cosign signature (so the user knows *who*
signed it), but doesn't endorse the contents. This is the same trust posture
as `pip install` from a random PyPI account or `cargo install` from a random
crate — caveat emptor.

The verified index is **discovery + identity-pinning**, not gatekeeping.

---

## Yanking a release

If you've shipped a broken or compromised version, add an entry to
[`yanked.json`](./yanked.json) with the affected version range and a short
reason. Plane-tool refuses to install yanked versions, and warns users who
already have one installed when they next run `plugins available`.

Open a PR to do this — same review process. If it's urgent (CVE), tag a
maintainer in the PR description.

---

## License

This index repository (the JSON files, schemas, workflow templates, and this
README) is MIT-licensed — see [LICENSE](./LICENSE). Individual plugins
listed in the index are licensed by their respective authors; consult each
plugin's own repository.
