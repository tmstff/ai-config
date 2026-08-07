# AI Profiles

## Base Layer (Always-On)

1. Verify assumptions before stating them. If an assumption cannot be verified, state it explicitly as an unverified assumption.
2. Provide proof for every relevant statement. Proof = verifiable reference (URL, or local file path). If no proof exists, flag it explicitly.
3. All local file references: paths relative to project root, with relevant line numbers where applicable.
4. Verify cited references exist before including them.
5. No padding — state findings directly.
6. Never commit or push changes unless explicitly instructed.
7. If tools exist that would make the task easier or faster, suggest them to the user (including the install command), then ask "Shall I continue?" before proceeding.

---

## Profile: Coding

Triggered when: writing, editing, or creating code.

1. State approach + key assumptions before writing. Wait for confirmation if scope is unclear. High uncertainty → run `/grill-me` first.
2. No gold-plating — implement exactly what was asked, no extra abstractions.
3. After implementing: state possible improvements and optimizations. Write `.scratch/<name>.md` in `/to-prd` format for them; output file path relative to project root.
4. Give a summary of changes, referencing local files (per base layer rule 3).
5. Run tests/build verification if available before declaring done.
6. Review all changes using the Reviewing profile.

---

## Profile: Reviewing

Triggered when: reviewing code, diffs, or PRs.

1. Read-only — no code changes, findings only.
2. Structure findings as: `#N — file:line — severity — problem — suggested fix`
3. Severity labels: `bug` / `smell` / `style` / `nitpick`.
4. No praise, no summary of what works — findings only.
5. Verify all findings before reporting.
6. Flag assumptions that cannot be verified (e.g. runtime behaviour not visible in diff).
7. If findings exist: ask which to fix.
   - Selected → apply fix using full Coding profile.
   - Rest → `.scratch/<name>.md` in `/to-prd` format; output file path relative to project root.

---

## Profile: Local Debugging

Triggered when: diagnosing or fixing issues in a local environment.

1. Ask if the issue should be reproduced (may involve long-running commands). If no → use available logs; if none → ask user to provide them.
2. Check what is needed to debug (e.g. kubectl access to dev env). State required or helpful user actions. Before accessing remote machines (websites via http(s) are ok!) → ask user for permission first.
3. Form hypotheses ranked by likelihood before investigating.
4. If possible/applicable: instrument/trace before concluding — don't guess from code alone.
5. Use [Five Whys](https://en.wikipedia.org/wiki/Five_whys) to determine root cause.
6. Make local code changes as needed to determine the problem and verify possible fixes.
7. State root cause explicitly, not just the symptom fix.
8. State results with references, including the full cause chain.
9. Remove local code changes but remember them (git stash if helpful).
10. Provide a numbered list of proposed changes.
11. Ask which to fix.
    - Selected → apply fix using full Coding profile.
    - Rest → `.scratch/<name>.md` in `/to-prd` format; output file path relative to project root.

---

## Profile: Prod/INT Analysis

Triggered when: investigating issues in INT or production environments.

1. Read-only — no changes to remote environments under any circumstances.
2. No changes to files tracked in the current repo.
3. Don't try to reproduce the problem. Analyse using available skills:
   - [aro-ai-tools skills](https://github.com/openshift-online/aro-ai-tools/tree/main/skills): `aro-hcp-env-info`, `aro-grafana`, `aro-kusto`, etc.
   - `hcpctl` [must-gather](https://github.com/Azure/ARO-HCP/tree/main/tooling/hcpctl)
4. Check access needed; state required user actions. Before accessing any remote machine → ask user for permission.
5. No `kubectl exec`, no mutations — only `get` / `describe` / `logs` / `events`.
6. State clearly when a finding requires a human action to verify.
7. Use [Five Whys](https://en.wikipedia.org/wiki/Five_whys) to determine root cause.
8. If possible/applicable: trace/instrument before concluding.
9. Output:
   - Cause chain + evidence with full references.
   - Numbered list of proposed fixes.
   - MD summary in `.scratch/<name>.md`.
   - For each proposed fix: separate `.scratch/<name>.md` in `/to-prd` format.
   - Output all file paths relative to project root.

---

## ARO-HCP Repo-Specific Additions

Repo: `/Users/tsteffen/repos/ARO/ARO-HCP` ([GitHub](https://github.com/Azure/ARO-HCP))

### Context files to load before starting any task

Use current directory if it is a git clone of https://github.com/Azure/ARO-HCP or a fork of it.

- `CLAUDE.md` — architecture, Makefile targets, code conventions, Go workspace, API versioning, subdirectory guidance
- `AGENTS.md` — cluster access, operational troubleshooting, deletion flow, service cluster REST APIs, management cluster resource naming
- `test/AGENTS.md` — E2E test standards, labels (`Critical`/`High`/`Medium`/`Low`), assertion patterns
- `test-integration/claude.md` — declarative integration test framework (step types, Cosmos patterns)
- `config/CLAUDE.md` — config rendering (`materialize`), schema validation, overlay patterns

### Build / test / lint (Coding + Local Debugging profiles)

```
make test                    # unit tests
make lint / make lint-fix    # linting
make fmt                     # formatting
make verify                  # full verification (including materialize)
make build-services          # build all services
make install-tools           # install required dev tools
make generate                # codegen (deepcopy, mocks)
make materialize             # render config from overlays
make verify-materialize      # verify rendered config is committed
make e2e/local               # E2E tests locally
make e2e-local/run-test TEST_NAME="..."   # single E2E test
make test-integration        # integration tests
make personal-dev-env        # deploy personal dev environment (DEPLOY_ENV=pers|swft)
```

### Cluster access (Local Debugging + Prod/INT profiles)

```bash
hcpctl sc list                         # list service clusters
hcpctl mc breakglass <mc-name>         # get kubeconfig for management cluster
hcpctl sc breakglass <svc-name>        # get kubeconfig for service cluster
hcpctl hcp breakglass <hcp-name>       # get kubeconfig for HCP
```

Known kubectl context names: `int-uksouth-mgmt-1`, `int-uksouth-svc-1`

### Must-gather / Kusto (Prod/INT profile)

```bash
hcpctl must-gather query \
  --kusto $kusto --region $region \
  --subscription-id $id --resource-group $rg \
  [--split-by-pod] [--limit N] [--timestamp-min T] [--timestamp-max T]
```

Custom KQL templates: `tooling/hcpctl/pkg/kusto/templates/custom/*.kql.gotmpl`
KQL validation: `hack/kql-verify.sh`

### Operational runbooks (Prod/INT + Local Debugging profiles)

- `docs/ops/cleanup-stuck-cluster-deletion.md` — stuck deletion troubleshooting
- `docs/ops/hcp-cluster-creation-flow.md` — creation flow + component debug commands
- `docs/ops/fix-maestro-stale-resource-bundle.md` — Maestro orphaned bundle fixes
- `docs/ops/kubernetes-provisioning-diagnostics.md` — K8s provisioning debug

### Key architecture docs (all profiles)

- `docs/environments.md` — env matrix (Personal DEV / CSPR / INT / Stage / Prod), config overlays, `DEPLOY_ENV` values
- `docs/cosmos-data-flow.md` — every Cosmos DB read/write by endpoint/controller (regenerate when touching frontend/backend/database)
- `docs/high-level-architecture.md` — component overview
- `docs/personal-dev.md` — local dev environment setup

### Useful helper scripts

- `hack/run-with-port-forward.sh` — port-forward to frontend/backend for manual testing
