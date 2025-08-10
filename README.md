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

| name               | default                     | description                                      |
|--------------------|-----------------------------|--------------------------------------------------|
| `mode`             | `declared`                  | `declared` \| `installed`                        |
| `ignore`           | `pip,setuptools,wheel,ruff` | Comma-separated packages to ignore               |
| `fail-if-outdated` | `true`                      | Fail if any outdated pins detected               |
| `max-unused`       | `0`                         | Max allowed unused declared deps (declared mode) |
| `format`           | `md`                        | `json` \| `md` \| `html`                         |
| `src`              | `.`                         | Source roots for import scan (space-separated)   |
| `pypi-ttl`         | `86400`                     | PyPI cache TTL (seconds)                         |
| `pypi-concurrency` | `8`                         | Parallel HTTP requests                           |
| `upload-artifact`  | `true`                      | Upload generated report as artifact              |
| `artifact-name`    | `animadao-report`           | Artifact name (**unique in matrix!**)            |
| `animadao-version` | `latest`                    | AnimaDao package version (`latest` or `x.y.z`)   |
| `python-version`   | `3.12`                      | Version for `actions/setup-python`               |

### Notes

- If you run this in a **matrix**, set unique `artifact-name` or merge later to avoid 409 conflicts.
- Add caching in the caller workflow to speed up uv:
  ```yaml
  - uses: actions/cache@v4
    with:
      path: ~/.cache/uv
      key: uv-${{ runner.os }}-${{ matrix.python-version }}-${{ hashFiles('uv.lock') }}
  ```

## One-liner GitHub Action

Use the official **composite Action** to run AnimaDao in one step.  
It writes a Markdown report to the **Job Summary**, optionally uploads artifacts, and can **fail** the job on policy
violations.

> Recommended pin: `uses: Absolentia/animadao-action@v1` (moving major tag).  
> For reproducibility you can pin to a specific tag, e.g. `@v1.0.1`.

### Minimal usage (declared mode)

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
    python-version: ${{ matrix.python-version }}
```

**Installed mode** (checks currently installed packages; no “unused” check):

```yaml
- name: AnimaDao (installed)
  uses: Absolentia/animadao-action@v1
  with:
    mode: installed
    fail-if-outdated: "true"
    ignore: pip,setuptools,wheel
    python-version: ${{ matrix.python-version }}
```

### Full example job (with cache + matrix)

```yaml
jobs:
  animadao:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        python-version: [ "3.10","3.11","3.12","3.13" ]
    steps:
      - uses: actions/checkout@v4

      # Speed up uv by caching
      - uses: actions/cache@v4
        with:
          path: ~/.cache/uv
          key: uv-${{ runner.os }}-${{ matrix.python-version }}-${{ hashFiles('uv.lock') }}

      - name: AnimaDao (declared)
        uses: Absolentia/animadao-action@v1
        with:
          mode: declared
          ignore: pip,setuptools,wheel,ruff
          fail-if-outdated: "true"
          max-unused: "0"
          format: md
          artifact-name: animadao-report-${{ matrix.python-version }}
          python-version: ${{ matrix.python-version }}
```

### Notes & tips

- **Artifacts in matrix:** artifact names must be **unique** per shard (
  `artifact-name: animadao-report-${{ matrix.python-version }}`), otherwise upload will fail with 409.
- **Report visibility:** the action appends the Markdown report to the **Job Summary**; HTML/JSON are attached as
  artifacts (if `upload-artifact: "true"`).
- **Policies:** relax or tighten as needed:
    - allow some unused deps: `max-unused: "1"`
    - ignore dev tools (e.g., `ruff`, `black`, `mypy`) via `ignore`.
- **Pre-release Python:** if you run on RCs (e.g., 3.14-rc), set `allow-prereleases: true` on your
  `actions/setup-python` step (outside of this action).
