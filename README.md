# AnimaDao Check — GitHub Action

Composite Action that runs **AnimaDao** to generate dependency health reports and enforce policies.

- Works with **uvx** (no install in the repo)
- Outputs a Markdown report into **Job Summary**
- Optionally uploads JSON/MD/HTML as an **artifact**
- Can **fail** the job on policy violations

## Usage

```yaml
- name: AnimaDao (declared)
  uses: Absolentia/animadao-action@v1
  with:
    mode: declared
    ignore: pip,setuptools,wheel,ruff
    fail-if-outdated: "true"
    max-unused: "0"
    format: md
    artifact-name: animadao-report-${{ matrix.python-version }}
```

Installed mode:

```yaml
- uses: Absolentia/animadao-action@v1
  with:
    mode: installed
    fail-if-outdated: "true"
    ignore: pip,setuptools,wheel
```

### Inputs

| name | default | description |
|------|---------|-------------|
| `mode` | `declared` | `declared` \| `installed` |
| `ignore` | `pip,setuptools,wheel,ruff` | Comma-separated packages to ignore |
| `fail-if-outdated` | `true` | Fail if any outdated pins detected |
| `max-unused` | `0` | Max allowed unused declared deps (declared mode) |
| `format` | `md` | `json` \| `md` \| `html` |
| `src` | `.` | Source roots for import scan (space-separated) |
| `pypi-ttl` | `86400` | PyPI cache TTL (seconds) |
| `pypi-concurrency` | `8` | Parallel HTTP requests |
| `upload-artifact` | `true` | Upload generated report as artifact |
| `artifact-name` | `animadao-report` | Artifact name (**unique in matrix!**) |
| `animadao-version` | `latest` | AnimaDao package version (`latest` or `x.y.z`) |
| `python-version` | `3.12` | Version for `actions/setup-python` |

### Notes
- If you run this in a **matrix**, set unique `artifact-name` or merge later to avoid 409 conflicts.
- Add caching in the caller workflow to speed up uv:
  ```yaml
  - uses: actions/cache@v4
    with:
      path: ~/.cache/uv
      key: uv-${{ runner.os }}-${{ matrix.python-version }}-${{ hashFiles('uv.lock') }}
  ```
