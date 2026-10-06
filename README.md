# validation-results

Portal JSON produced by the Geant4 validation pipeline, one directory per test
repository. Each test's directory is overwritten in full by its own CI run —
history lives in this repo's git log and in each merge commit's message, not
in parallel per-run folders.

## Layout

```
<test-name>/
  results.json        # all plots for this test, as one array (portal-facing)
  plots/<md5>.json     # same plots, one object per file (legacy import format)
  provenance.json      # where this run came from (see below)
```

`<test-name>` matches the name used in `ci-workflows`'
[`tests.json`](https://github.com/G4Med-test/ci-workflows/blob/main/tests.json)
(e.g. `LowEFrag`, `CCCStest`). `results.json` and `plots/` keep the original
portal schema (`article`, `mctool`, `testName`, `metadata`, `plotType`,
`histogram`/`chart`) — see
[ci-workflows/README.md](https://github.com/G4Med-test/ci-workflows#validation-and-portal-json)
for the schema itself.

## `provenance.json`

```json
{
  "test_repo": {"url": "https://github.com/G4Med-test/LowEFrag", "sha": "<commit>"},
  "container_repo": {"url": "https://github.com/G4Med-test/geant4-alma9",
                      "sha": "<commit>", "geant4_commit": "<Geant4 source commit>"},
  "image": "oras://ghcr.io/g4med-test/lowefrag:<sha>",
  "geant4_version": "11.3.2"
}
```

- `test_repo`: the test repository and commit whose macros and parser produced
  these plots.
- `container_repo`: the `geant4-alma9` recipe commit the image was built from,
  and the Geant4 source commit it compiled — read from the image's own OCI
  labels (`org.opencontainers.image.revision`, `org.g4med.geant4.commit`),
  not re-derived or guessed.
- `image`: the exact oras:// reference run for this result.

If a field is missing (`null`), the pipeline couldn't determine it — e.g. a
local/manual run outside GitHub Actions, or an image built before these
labels existed.

## How results get here

`ci-workflows`' [`run-validation-padova.yml`](https://github.com/G4Med-test/ci-workflows/blob/main/.github/workflows/run-validation-padova.yml)
runs the `publish-results` job after a successful validation: it replaces the
test's directory wholesale and opens a pull request here (branch
results/<test-name>), merged with squash as soon as it's mergeable. A second run of the same test
before the first PR merges updates that same PR instead of opening a
competing one; the workflow also serializes same-test runs so two of them
never race to merge conflicting content.


Nothing here is meant to be edited by hand — changes should only ever arrive
through that PR flow.
