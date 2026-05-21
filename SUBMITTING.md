# Submitting a plugin to the OpenScientist index

Two steps: **publish** your plugin to a public OCI registry, then **submit**
a PR to add an entry here.

## 1. Publish

### Layout of your plugin repo

```
your-plugin-repo/
├── plugin.json              # OpenScientist manifest (REQUIRED)
├── README.md                # surfaces in the install dialog
├── icon.png                 # ~128x128 (REQUIRED for the index)
├── bin/                     # optional — CLI tools
├── server/                  # optional — long-running HTTP
├── ui/                      # optional — iframe UI
└── .github/workflows/
    └── release.yml          # copy from this index repo's templates/release.yml
```

`plugin.json` must validate against the [OpenScientist plugin manifest
schema](https://github.com/fuzzy-dynamics/gitclines/blob/master/plugin_architecture.md#3-manifest--pluginjson).

### The release workflow

Copy [`templates/release.yml`](./templates/release.yml) verbatim into
`.github/workflows/release.yml` in your plugin repo. The only customization
is the `plugin_files` glob if your layout differs.

The workflow:

- Fires on every `v*.*.*` tag.
- Packs `plugin.json` + your declared assets into a gzipped tar.
- Pushes via `oras` to `ghcr.io/<your-namespace>/<repo-name>:<version>`.
- Signs the artifact with `cosign` using Sigstore keyless signing —
  no API keys, no PAT, the GitHub Actions OIDC identity is the signer.

> ⚠️ **The `permissions:` block** matters. The workflow needs
> `packages: write` (to push to ghcr.io) **and** `id-token: write` (to
> request a Sigstore OIDC token). If either is missing, signing fails.

### Verify your first release

After your first tag push, your Action should succeed and you should see:

```
ghcr.io/<you>/<plugin>:0.1.0          ← OCI artifact
ghcr.io/<you>/<plugin>:sha256-...sig  ← cosign signature
```

Confirm both with:

```bash
cosign verify ghcr.io/<you>/<plugin>:0.1.0 \
  --certificate-identity-regexp 'https://github.com/<you>/<plugin>/.github/workflows/release.yml@refs/(heads|tags)/.*' \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com
```

If this prints `Verified OK` you're ready to submit.

## 2. Submit

Open a PR against this repo (`fuzzy-dynamics/OpenScientist-plugins`) adding
**one** entry to `index.json`:

```jsonc
{
  "id": "witsoc-extras",
  "publisher": "alice",
  "ref": "ghcr.io/alice/witsoc-extras",
  "trusted_identity": "https://github.com/alice/witsoc-extras/.github/workflows/release.yml",
  "trusted_oidc_issuer": "https://token.actions.githubusercontent.com",
  "repository_url": "https://github.com/alice/witsoc-extras",
  "homepage_url": "https://alice.dev/witsoc-extras",
  "displayName": "Witsoc Extras",
  "description": "Extra proof tactics for the witsoc plugin.",
  "tags": ["math", "witsoc", "proof"],
  "license": "MIT"
}
```

The `trusted_identity` is the workflow URL **prefix** — *no* `@refs/...`
suffix. The verifier appends `@refs/(tags|heads)/.+` automatically so a
single index entry validates every tagged release without having to be
re-PR'd.

### Fields

| Field | Required | Meaning |
|---|---|---|
| `id` | ✓ | Globally-unique slug (kebab-case). Becomes the install dir name `~/.openscientist/plugins/<id>/`. |
| `publisher` | ✓ | Your GitHub handle or org name (informational). |
| `ref` | ✓ | OCI repository reference *without tag* — plane-tool appends `:latest` for install or `:<version>` if user pinned. |
| `trusted_identity` | ✓ | URL prefix of the GitHub Actions workflow that signs releases (no `@refs/...`). Verifier auto-appends the ref pattern. Must match the workflow path under your repo exactly. |
| `trusted_oidc_issuer` | optional | OIDC issuer. Default `https://token.actions.githubusercontent.com` (GitHub Actions). Change only if you sign from a non-GitHub CI. |
| `repository_url` | ✓ | https URL to your plugin's source code repo. |
| `homepage_url` | optional | Marketing URL. |
| `displayName` | ✓ | Human-readable name for the install dialog. |
| `description` | ✓ | One sentence. Shown next to the name in `plugins available`. |
| `tags` | optional | Searchable tags. Lowercase kebab-case. |
| `license` | recommended | SPDX identifier (e.g., `MIT`, `Apache-2.0`). |

### What the reviewer checks

- JSON schema (CI auto-fails malformed PRs).
- `ref` is reachable (`oras manifest fetch` returns 200).
- `cosign verify` succeeds against the `trusted_identity`.
- `plugin.json` from the latest tag parses and declares reasonable
  capabilities (no surprise `network.outbound` without justification).
- README is clear and reflects the plugin's actual behavior.
- Icon is present and reasonable.
- No name collision with existing entries.

If any check fails, the reviewer leaves a comment with what needs fixing.

### Updating an existing entry

If you've already shipped — just push a new tag in your plugin repo. The
index *doesn't* track versions; plane-tool always resolves the latest tag at
the OCI level. The only time you need to PR the index again is to change
metadata (description, tags, displayName) or to **yank** a bad release (see
[`yanked.json`](./yanked.json)).

## Withdrawing a plugin

Open a PR removing your entry from `index.json`. Already-installed users
continue to function (their local copy is unaffected), but no new installs
are possible via the verified track. Users can still install directly via
`--allow-untrusted` if your ghcr.io artifact remains public.

If you also want to prevent existing users from updating to specific
versions, add a `yanked.json` entry covering those versions.
