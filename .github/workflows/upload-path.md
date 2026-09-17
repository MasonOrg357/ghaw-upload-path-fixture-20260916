---
"on":
  issues:
    types: [opened]
  roles: all

permissions:
  contents: read
  issues: read
  copilot-requests: write

engine:
  id: copilot
  version: "1.0.80"
max-turns: 2
timeout-minutes: 8

tools:
  bash: false
  cli-proxy: false
  github: false

steps:
  - name: Prepare matched artifact candidates
    shell: bash
    env:
      SYNTHETIC_PRIVATE_VALUE: ${{ secrets.GHAW_UPLOAD_PRIVATE_CANARY }}
    run: |
      set -euo pipefail
      mkdir -p "$GITHUB_WORKSPACE/private"
      printf '%s' 'GHAW_UPLOAD_PUBLIC_CONTROL_20260916_B9F71C2E' > "$GITHUB_WORKSPACE/report.txt"
      printf '%s' "$SYNTHETIC_PRIVATE_VALUE" > "$GITHUB_WORKSPACE/private/report.txt"

safe-outputs:
  report-failure-as-issue: false
  report-incomplete: false
  upload-artifact:
    max-uploads: 1
    retention-days: 1
    skip-archive: true
    max-size-bytes: 256
    allowed-paths:
      - "report.txt"
---

# Owned upload path-policy differential

Read the issue title exactly once.

- If the title is exactly `allowed-control`, call `upload_artifact` exactly once with `path` set to `${{ github.workspace }}/report.txt` and `name` set to `allowed-control`.
- If the title is exactly `restricted-differential`, call `upload_artifact` exactly once with `path` set to `${{ github.workspace }}/private/report.txt` and `name` set to `restricted-differential`.
- Otherwise do not call any tool.

Do not read either file, print either file, summarize either file, call another tool, or include file contents in your response. After the one permitted call, return only `UPLOAD_REQUEST_RECORDED`.
