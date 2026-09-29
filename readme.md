# mise: multi-version tool updates overwrite the wrong lockfile entry

## Current behavior

`mise.toml` configures three Node versions:

```toml
[tools]
node = ["24", "22", "20"]
```

`mise.lock` has one entry per version, each tagged with `specifiers`, in the order 20, 24, 22.

Renovate takes the locked version from the first `[[tools.node]]` entry, without matching `specifiers` against the configured selector. Selector `24` gets paired with `20.20.2`, so Renovate opens a major update `20.20.2` -> `v24.x`. The Node 20 entry in `mise.lock` is then overwritten with Node 24, the `24` specifier is merged into it, and the existing Node 24 entry loses its specifier. Node 20 disappears from the lockfile.

## Expected behavior

Each configured selector is matched to the lockfile entry whose `specifiers` contain it. Selector `24` resolves to `24.21.0`. No PR should overwrite the Node 20 entry.

## Reproduction

The [`renovate` workflow](.github/workflows/renovate.yml) runs Renovate CLI 44.119.0 against this repository with `LOG_LEVEL=debug`.

Result on Renovate 44.119.0:

- PR: [#1 Update dependency node to v24](https://github.com/matchai/renovate-repro-mise-lock-specifiers/pull/1), `20.20.2` -> `v24.21.0` (major)
- Debug log: [workflow run 36562298518](https://github.com/matchai/renovate-repro-mise-lock-specifiers/actions/runs/36562298518)

The extracted dependency pairs the `24` selector with the Node 20 entry:

```json
"currentValue": "24",
"lockedVersion": "20.20.2",
"rangeStrategy": "update-lockfile",
"isLockfileOnly": true
```

The `mise lock node` artifact step then processes `node@24.21.0, node@22.23.3, node@24.21.0`, so Node 20 is dropped from `mise.lock`.

## Links

- Discussion: TODO
- Proposed fix: https://github.com/renovatebot/renovate/pull/46407
