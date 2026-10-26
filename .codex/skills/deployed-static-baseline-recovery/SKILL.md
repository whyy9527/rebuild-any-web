---
name: deployed-static-baseline-recovery
description: |
  Recover a version-pinned static runtime baseline during a project split or rebuild
  when the configured upstream asset URL returns 404 but the live deployment may
  still serve the exact content. Use when ignored/private generated site assets are
  part of deployment and a Git-only extraction would omit them; not for restoring
  credentials, private user data, raw provider payloads, or an unverified neighboring release.
---

# Deployed Static Baseline Recovery

## Use When

- A project is being split from a larger repository and the app runs from an ignored generated directory such as `private/site`.
- `source-build.json` or another pinned upstream URL no longer serves the release, while the current service may still expose its static files under a read-only content path.
- A cold clone needs the same deployed baseline for its local server or preview package.

The target behavior is a reproducible, version-matched local baseline. A copy that merely loads, or assets from a nearby upstream version, is still a failure.

## Failure Mechanism

Git history and the live package are different file sets. A Git-only split can omit generated runtime files that were intentionally ignored. Conversely, recursively packaging source directories can include ignored or unreviewed files. A live endpoint returning 200 proves availability, not identity: the restored content must match a trusted build identifier and per-file integrity data.

## Procedure

1. Trace one deployed file end to end: pinned build configuration, preparation code, server route, release packer, and validation. Read implementations rather than inferring scope from directory names. Record which files are tracked source, generated public runtime assets, and private evidence.
2. Find the exact runtime content route in the current deployment and verify it with read-only requests. Check the public release/build identity and compare it with the pinned configuration. Do not change DNS, publish, install certificates, or write to production as part of recovery.
3. Establish a trusted expected file list and digest for that exact build. Reuse an existing upstream/provenance/deployment manifest when available. Otherwise derive one from a previously reviewed local baseline that is independently tied to the pinned build. Do not generate expected hashes from the production response itself and then treat the same response as verified.
4. Model the file set from the actual packer. Include only static runtime files that the deployment serves. Keep public raw inputs and derived runtime outputs distinguishable when useful. Keep credentials, signed URLs, private captures or recordings, private provider transport requests/responses, and unrelated local state out of the recovery bundle. Published application data remains part of the runtime closure when the app needs it. Never copy the whole ignored `private/` directory.
5. Fetch every expected file from the production content route into a new staging directory. Validate relative paths remain inside staging, HTTP success, byte count, and SHA-256 before installing anything. Fail closed on a missing file or any mismatch; preserve the failure list and stop. Do not fill a missing item from an adjacent build, another cache, or an unverified local copy.
6. Once the complete set and embedded build configuration match, atomically install the staged tree into the ignored runtime location. Preserve or archive the previous local tree so a failed switch can roll back. Make production restoration an explicit source option; do not silently fall back to production whenever the pinned upstream fetch fails.
7. Validate from a clean local runtime directory: run the restoration command, start the real local server, make a representative read-only request, and use the repository's meaningful tests or headless browser smoke. Confirm the command loads no baseline from a sibling worktree or machine-global cache.
8. Keep deployment packaging closed: select source files from `git ls-files` (and reject uncommitted release inputs) and include the generated baseline only through its explicit reviewed file rules. Do not recursively add every file found under `app/`, `deploy/`, or `private/`.

## Verification

A recovery is verified only when the exact pinned build is identified, every file in the actual deployment closure has matching expected bytes and hash, a clean local restoration succeeds, and the restored server is exercised. Report the verified set, failed or excluded files, and any rights limits. Static HTTP status checks or a root-page screenshot alone do not prove complete recovery.

The [S16 recovery implementation](https://github.com/xinxiayiyan/s16/blob/main/tools/restore_baseline.py), [production content manifest](https://github.com/xinxiayiyan/s16/blob/main/provenance/production-content-manifest.json), and [preparation entry point](https://github.com/xinxiayiyan/s16/blob/main/tools/prepare.py) are a verified example: the original fixed upstream release was unavailable, while the current read-only content route yielded the exact locally expected release file set. This example verifies that case only; automatic discovery and recovery success across other deployment layouts remain untested.

## High-Risk Actions

- Do not write to production, change routing/DNS, or publish a release to make recovery easier.
- Do not treat a 200 response, matching directory name, or build label alone as proof of byte identity.
- Do not silently select a nearby version or synthesize missing files.
- Do not copy an entire ignored/private directory or add raw private evidence, credentials, signed URLs, recordings, or model call payloads to Git.
- Do not package untracked or ignored source files through a broad recursive scan.
