<!--
Thanks for submitting an OpenScientist plugin!

If you're adding a new plugin to the verified index, fill out the checklist
below and replace the placeholders. If you're yanking a release or fixing
metadata, just say so in the description and skip the rest.
-->

## What this PR does

- [ ] Add a new plugin to `index.json`
- [ ] Update metadata for an existing plugin
- [ ] Yank one or more versions in `yanked.json`
- [ ] Other (please describe)

## Plugin info (skip for non-add PRs)

- **id:**            `<your-plugin-id>`
- **publisher:**     `<your-github-handle-or-org>`
- **OCI ref:**       `ghcr.io/<you>/<plugin>`
- **Trusted ident:** `https://github.com/<you>/<plugin>/.github/workflows/release.yml`  (workflow URL only, no `@refs/...`)
- **Repo:**          `https://github.com/<you>/<plugin>`
- **Latest tag:**    `v0.1.0`

## Author checklist

- [ ] My plugin is published to a public OCI registry (`ghcr.io/...`).
- [ ] I copied the reference [`release.yml`](../templates/release.yml) into
      my plugin repo and at least one tag has been successfully published.
- [ ] `cosign verify` of my `:latest` tag succeeds locally against my
      `trusted_identity`.
- [ ] My `plugin.json` declares only the capabilities the plugin actually
      uses (no unused `network.outbound`, etc.).
- [ ] My README explains what the plugin does, how to use it, and lists any
      external services it depends on.
- [ ] My plugin has an `icon.png` (~128×128).

## Reviewer notes

CI runs schema + signature checks automatically. Reviewers should
additionally verify:

- [ ] README is clear and matches the plugin's actual behavior
- [ ] Capability list is reasonable
- [ ] No name squat / namespace collision
- [ ] License is declared (or noted as unlicensed)
- [ ] Icon and screenshot (if any) are appropriate
