# npm wrapper (npm → pnpm CI shim)

CI prepends this directory to `PATH` so bare `npm` invocations are handled by
[`npm.py`](npm.py) (via [`npm`](npm) on Unix or [`npm.bat`](npm.bat) on
Windows).

## When translation happens

Translation runs only when `pnpm-lock.yaml` exists in the project directory:

- Current working directory, or
- The path given by `--prefix <dir>` (resolved relative to cwd).

Otherwise the wrapper delegates to the real `npm` on `PATH` (with this directory
removed to avoid recursion).

## Command mapping (pnpm projects)

| npm                        | pnpm                                       |
| -------------------------- | ------------------------------------------ |
| `install`                  | `install`                                  |
| `ci`                       | `install --frozen-lockfile`                |
| `install <pkg>`            | `add <pkg>`                                |
| `run` / `run-script`       | `run` (standalone `--` tokens are dropped) |
| `version`                  | `version`                                  |
| `cache clean --force`      | `store prune`                              |
| `audit`                    | `audit`                                    |
| `--prefix <dir>`           | `--dir <dir>`                              |
| `--silent`                 | `--reporter=silent`                        |
| `--no-fund` / `--no-audit` | omitted                                    |

`npm install` / `npm ci` remove an existing npm-style `node_modules` tree (not a
pnpm `.pnpm` layout) before installing. `package-lock.json` is left in place.

## Passthrough (always real npm)

- No `pnpm-lock.yaml` in the target project directory
- `npm install -g` / `--global`
- `BRAVE_NPM_WRAPPER=0` (or `false` / `no` / `off`)

## CI setup

After pnpm is installed on the agent (see `devops/packer/shared/linux_base.sh`):

```bash
export PATH="/path/to/src/brave/build/npm_wrapper:${PATH}"
```

On Windows, ensure `npm.bat` from this directory precedes `npm.ps1` on `PATH`
(build images already remove `npm.ps1` where needed).

## Tool versions

`npm --version` reports the pnpm version when the wrapper is active for a pnpm
project (see Jenkins `listToolVersions`).

## Tests

```bash
cd brave/build/npm_wrapper
python3 -m unittest npm_wrapper_test -v
```

## Related

[`script/audit_deps.py`](../script/audit_deps.py) runs `pnpm audit --json` when
`pnpm-lock.yaml` is present in an audited directory.
