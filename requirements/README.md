## Python dependency logic

In this folder, the `.in` files, in conjunction with the `../pyproject.toml` file (in the root of the repo) are used to generate the pinned requirements as `.txt` files.

To alter the pinned dependency, you can edit/alter the `.in` and `pyproject.toml` files, and then run the following command:

```bash
./scripts/uv-pip-compile.sh
```
:::warning
The pinned dependencies are based on the `current` version of python supported in Superset.
Output of `./scripts/uv-pip-compile.sh` may vary slightly based on the python version you are using to run the command.
Check the `pyproject.toml` file for the current version of python supported.
:::

This will generate the pinned requirements in the `.txt` files, which will be used in our CI/CD pipelines and in the Docker images.

We recommend to everyone in the community to use the pinned requirements in their local development environments, to ensure consistency across different environments, though we don't force requirements as part of our python package semantics to allow flexibility for users to install different versions of the dependencies if they wish.

Note that `development.txt` is a superset of what's in `base.txt`, and all version numbers for shared library should fully match at all times. `translations.txt` is meant as a supplemental file to be used in conjunction with the other requirements files, and is not meant to be used standalone.

## Security remediation notes

### Flask-CORS 6.x

- Manifest pin in `../pyproject.toml`: `flask-cors>=6.0.0, <7.0`.
- Compiled pin in `base.txt` and `development.txt`: `flask-cors==6.0.2`.
- Safe range: `>= 6.0.0` (see CHANGELOG/6.0.0.md entry for the original bump).
- Scanner check: `pip-audit -r requirements/base.txt` reports no `flask-cors` finding. The manifest, the `.in` chain (transitive via `pyproject.toml`), and the compiled `.txt` outputs are aligned and within the safe range.
