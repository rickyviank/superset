<!--
  Licensed to the Apache Software Foundation (ASF) under one
  or more contributor license agreements.  See the NOTICE file
  distributed with this work for additional information
  regarding copyright ownership.  The ASF licenses this file
  to you under the Apache License, Version 2.0 (the
  "License"); you may not use this file except in compliance
  with the License.  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing,
  software distributed under the License is distributed on an
  "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
  KIND, either express or implied.  See the License for the
  specific language governing permissions and limitations
  under the License.
-->

# Superset Frontend Security Audit Notes

This file tracks recurring frontend supply-chain audit findings and their
current status in this fork. Each entry should record what was checked, the
scanner evidence, and the rollback procedure so the same finding does not
require a re-investigation from scratch.

## node-fetch (transitive)

- **Status:** Safe. No remediation required at this time.
- **Last verified:** 2026-05 against `superset-frontend/package-lock.json` on
  `master`.

### Resolved versions

`node-fetch` has a single resolution in the lockfile:

```
superset@0.0.0-dev /superset-frontend
└─┬ google-auth-library@10.6.2
  └─┬ gaxios@7.1.4
    └── node-fetch@3.3.2
```

Reproduce with:

```bash
cd superset-frontend
npm ci
npm ls node-fetch --all
```

### Scanner evidence

`npm audit --omit=dev` reports zero `node-fetch` advisories. The 3.x line of
`node-fetch` has no known CVEs; the historical advisory
[GHSA-r683-j2x4-v87g](https://github.com/advisories/GHSA-r683-j2x4-v87g)
applies only to `node-fetch < 2.6.7`, which is not present in the tree.

Reproduce with:

```bash
cd superset-frontend
npm audit --omit=dev --json \
  | jq '[(.vulnerabilities // {}) | to_entries[] | select(.key | test("node-fetch"; "i"))]'
# => []
```

### Why no `overrides` block

Per the project guideline of preferring parent upgrades over forced overrides,
no override is added: the only parent (`gaxios@7.1.4`, latest at audit time)
already requires `node-fetch@^3.3.2`. Pinning a transitive version that is
already on the safe range would be a hard-coded workaround rather than a
general-purpose fix and would mask future upstream regressions.

### Rollback

This change is documentation only — no dependency was bumped and no
`package.json` / `package-lock.json` content was modified. Reverting the
commit restores the previous state with no behavioral or build impact.

### Re-checking on future audits

If a future scan reports a `node-fetch` finding:

1. Re-run the commands above to confirm the resolved version(s).
2. If a `node-fetch < 2.6.7` copy reappears, identify the new parent with
   `npm ls node-fetch --all` and bump that parent first.
3. Only add an entry to the `overrides` block in
   `superset-frontend/package.json` if no parent upgrade is viable; document
   the reason in this file.
