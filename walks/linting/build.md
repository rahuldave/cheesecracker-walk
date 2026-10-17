<!-- Made by the walk skill's render.py from the facts of the walk. Do not edit it by hand: run just walk-update again. -->

# The formatter and the linter of cheesecracker

**Build it yourself.** Every command to run and every change to make by hand, to make the project yourself. [Read the tutorial](read.md) is the other way to read it.

Each part ends with a section called **What changed**. It lists every file that the part changes, with a link to the file, and then the whole diff of each file, folded. A lockfile and a binary file have a link alone. The text above it explains the change. What changed is what you check your files against.

In Build it yourself, a file that a command makes says which command. A diff that is open is a patch. Apply it with the command in the text, or make the same change in your editor.

We check the code of cheesecracker, the API server of the Cheese App. Code that works can still be hard to read, and
a reader can miss a bug in it. A formatter writes the code in one layout, and a linter checks it against rules. ruff is
both. pre-commit then runs the checks before every `git commit`. This tutorial takes these parts out of the whole
tutorial of cheesecracker.

We make the project in moves. A move is one small change, made by one patch or by one command. You make each move,
and the page checks your files against it.

The first sections build the code to check, a function, a server and their tests. Some changes of the whole tutorial are not in this one. After such a gap, the next section says what those changes
did, and starts from the code that they left. It first runs `just setup`, which installs the packages of that code.

## Get the project

You build the project in a clone of your own, on a branch `mine` that starts where this walk starts. Run the commands of these pages in its top folder.

```bash
git clone https://github.com/rahuldave/cheesecracker
cd cheesecracker
git switch -c mine cf50a6ef5045
```

Then add the command `git check`, which compares your files with the project's after each move, and leaves what `git add` has staged as it was. It compares `pyproject.toml` and `uv.lock` by the names of their packages, so another version of a package does not count as a difference. A small Python program, `package_names.py`, reads each of these files as data, and uv runs it. These commands write the program into the folder `.git`, and tell git to show each file as the program prints it.

```bash
cat > "$(git rev-parse --git-common-dir)/package_names.py" <<'PY'
"""Print a pyproject.toml or a uv.lock without the versions of its packages, so that git check compares their names.

    python package_names.py pyproject.toml FILE
    python package_names.py uv.lock FILE

A pyproject.toml keeps all it says, with each requirement cut to its name, its extras, its address and its marker. A
uv.lock keeps the names that the project's own packages ask for, and the members of the workspace. The rules are those
of timewalk's package_names. The result is JSON, one item a line. A file that is not TOML is printed as it is.
"""

import json
import re
import sys
import tomllib

# A requirement such as "pytest>=8.3.4", "Uvicorn[standard] ; python_version < '3.12'" or "x @ git+https://...": its name,
# its extras, what comes between (the versions, or "@" and an address), and its marker after the ";".
REQUIREMENT = re.compile(r"^\s*([A-Za-z0-9][A-Za-z0-9._-]*)\s*(\[[^\]]*\])?([^;]*)(?:;(.*))?$", re.S)


def package_names(
    path: str,  # The file's path in the project; its name says how to read it
    data: bytes,  # Its contents
) -> object | None:  # What it says without the versions of packages, to compare; None if it is not a file of packages
    """Read a pyproject.toml or a uv.lock with the versions left out, as timewalk's package_names does."""
    name = path.rsplit("/", 1)[-1]
    if name not in ("pyproject.toml", "uv.lock"):
        return None
    try:
        doc = tomllib.loads(data.decode("utf-8"))
    except (UnicodeDecodeError, tomllib.TOMLDecodeError):
        return None

    def cut(requirement: object) -> object:   # "pytest>=8.3.4" is pytest; a table, such as { include-group = "test" }, stays
        found = REQUIREMENT.match(requirement) if isinstance(requirement, str) else None
        if not found:
            return requirement
        extras = sorted(e.strip().lower() for e in (found.group(2) or "[]")[1:-1].split(",") if e.strip())
        between, marker = found.group(3).strip(), " ".join((found.group(4) or "").split())
        address = between[1:].strip() if between.startswith("@") else ""   # a package from an address: the address counts
        return (re.sub(r"[-_.]+", "-", found.group(1)).lower() + (f"[{','.join(extras)}]" if extras else "")
                + (f" @ {address}" if address else "") + (f"; {marker}" if marker else ""))

    def names(value: object) -> list:   # every name = "..." in a table, at any depth
        if isinstance(value, dict):
            return [*([value["name"]] if isinstance(value.get("name"), str) else []), *(n for v in value.values() for n in names(v))]
        return [n for v in value for n in names(v)] if isinstance(value, list) else []

    if name == "uv.lock":
        own = [p for p in doc.get("package", []) if isinstance(p, dict) and isinstance(p.get("source"), dict)
               and {"editable", "virtual"} & set(p["source"])]
        return {"requires-python": doc.get("requires-python"), "manifest": sorted(set(names(doc.get("manifest", {})))),
                "own": {p.get("name"): sorted(set(names({k: v for k, v in p.items() if k not in ("name", "version", "source")})))
                        for p in own}}
    lists = [("project", "dependencies"), ("build-system", "requires"), ("tool", "uv", "dev-dependencies")]
    groups = [("project", "optional-dependencies"), ("dependency-groups",)]

    def at(keys: tuple) -> object:   # the table or list at a path of keys, or None
        value = doc
        for key in keys:
            value = value.get(key) if isinstance(value, dict) else None
        return value

    for keys in lists:
        if isinstance(at(keys), list):
            at(keys[:-1])[keys[-1]] = sorted(map(cut, at(keys)), key=repr)
    for keys in groups:
        table = at(keys)
        for group, wanted in (table.items() if isinstance(table, dict) else []):
            if isinstance(wanted, list):
                table[group] = sorted(map(cut, wanted), key=repr)
    return doc


def main() -> int:  # The exit code
    """Print a file as package_names reads it, or as it is when it is not TOML."""
    kind, file = sys.argv[1], sys.argv[2]
    with open(file, "rb") as handle:
        data = handle.read()
    found = package_names(kind, data)
    if found is None:
        sys.stdout.buffer.write(data)
    else:
        print(json.dumps(found, indent=1, sort_keys=True, ensure_ascii=False, default=str))
    return 0


if __name__ == "__main__":
    sys.exit(main())
PY
echo '/pyproject.toml diff=pyproject' >> .git/info/attributes
echo '/uv.lock diff=uvlock' >> .git/info/attributes
git config diff.pyproject.textconv 'uv run --quiet --no-config --no-project --python ">=3.11" python "$(git rev-parse --git-common-dir)/package_names.py" pyproject.toml'
git config diff.uvlock.textconv 'uv run --quiet --no-config --no-project --python ">=3.11" python "$(git rev-parse --git-common-dir)/package_names.py" uv.lock'
git config alias.check '!f() { i="$(git rev-parse --git-path check-index)" && { cp -p "$(git rev-parse --git-path index)" "$i" 2>/dev/null || rm -f "$i"; } && export GIT_INDEX_FILE="$i" && git add -A && git diff --cached "$1" -- ":/"; }; f'
```

<a id="step-00"></a>

## 0 The project

This walk goes through [rahuldave/cheesecracker](https://github.com/rahuldave/cheesecracker) on GitHub. This part is where the project starts, at `step-00`. [The code of this part](https://github.com/rahuldave/cheesecracker/tree/step-00)

The project starts empty, with a `pyproject.toml`, a lockfile and a `justfile`.

```bash
just setup
```

Two tools run everything in this project, uv and just.

[uv](https://docs.astral.sh/uv/) manages a Python project. It installs Python, and the packages that the project
needs, into the folder `.venv` of the project. `pyproject.toml` lists the packages, and `uv.lock` holds the exact
version of each. So every machine gets the same versions.

`uv run CMD` runs a command with that Python and those
packages. uv is written in Rust by Astral. It does the work of several older Python tools, such as pip, which installs packages.

[just](https://just.systems/) runs the commands of a project by short names. Each name is a recipe in the file
`justfile`, with the commands that it runs. `just NAME` runs a recipe, and `just` alone lists them. A recipe keeps
a long command in one place, so everyone on the project runs it by the same short name.

The project is empty. It has a `pyproject.toml` with no dependencies, an empty `uv.lock`, a `README.md`, a
`.gitignore`, and a `justfile` with one recipe, `setup`. `just setup` runs `uv sync --locked`. The command makes the
folder `.venv`, with Python 3.12 and every package of `uv.lock` in it.

Run it a second time. With nothing new in `uv.lock`, it changes nothing.

```bash
just setup
```

uv prints `Resolved 1 package` and `Checked`, and no line `Creating virtual environment`.

Two settings of `pyproject.toml`, in the table `[tool.uv]`, matter for the whole project. `package = false` says that
cheesecracker is an application. uv installs its dependencies, and does not install the project as a package.
`default-groups = "all"` makes `uv sync` and `uv run` install every group of dependencies, such as the tools of the
tests.

```bash
just --list
```

`just --list` prints the one recipe, `setup`, with its comment.

Check your files against this part. The command prints nothing when they match, and otherwise shows what differs:

```bash
git check cf50a6ef5045
```

If you are stuck, this command puts the files of this part in place of yours. A file that you made yourself stays:

```bash
git fetch --quiet --tags --force origin; git rm -rq --cached --ignore-unmatch -- :/ && git restore --source step-00 --staged --worktree -- :/
```

### What changed

- <a id="step-00-.gitignore-a5cc29"></a>`.gitignore`: the environment, the caches of the tools, and the secrets, which git never takes. New, 21 lines. [The file](https://github.com/rahuldave/cheesecracker/blob/cf50a6ef5045e08ca727aeaffdb710fad2cae096/.gitignore).
- <a id="step-00-.python-version-7696e9"></a>`.python-version`: the version of Python that uv installs, 3.12. New, 1 line. [The file](https://github.com/rahuldave/cheesecracker/blob/cf50a6ef5045e08ca727aeaffdb710fad2cae096/.python-version).
- <a id="step-00-README.md-8ec9a0"></a>`README.md`: what the project is, and how to set it up. New, 14 lines. [The file](https://github.com/rahuldave/cheesecracker/blob/cf50a6ef5045e08ca727aeaffdb710fad2cae096/README.md).
- <a id="step-00-justfile-424f0b"></a>`justfile`: the recipe `setup`, and `default`, which lists the recipes. New, 12 lines. [The file](https://github.com/rahuldave/cheesecracker/blob/cf50a6ef5045e08ca727aeaffdb710fad2cae096/justfile).
  - look at `uv sync --locked`
- <a id="step-00-pyproject.toml-5d07e7"></a>`pyproject.toml`: the name of the project, no dependencies yet, and the table `[tool.uv]`. New, 13 lines. [The file](https://github.com/rahuldave/cheesecracker/blob/cf50a6ef5045e08ca727aeaffdb710fad2cae096/pyproject.toml).
  - look at `default-groups = "all"`
- <a id="step-00-uv.lock-8356d8"></a>`uv.lock`: the lockfile, with the project and no packages yet. New, 8 lines. [The file](https://github.com/rahuldave/cheesecracker/blob/cf50a6ef5045e08ca727aeaffdb710fad2cae096/uv.lock).

<a id="step-00-.gitignore-a5cc29-diff"></a>

<details><summary>The diff of <code>.gitignore</code></summary>

```diff
@@ -0,0 +1,21 @@
+# Python
+__pycache__/
+*.py[cod]
+.venv/
+
+# What the tests and the coverage write
+.pytest_cache/
+.coverage
+htmlcov/
+.ruff_cache/
+
+# Secrets: never commit them
+secrets/
+.env
+*.key
+*.pem
+
+# The editor and the system
+.vscode/
+.idea/
+.DS_Store
```

</details>

<a id="step-00-.python-version-7696e9-diff"></a>

<details><summary>The diff of <code>.python-version</code></summary>

```diff
@@ -0,0 +1 @@
+3.12
```

</details>

<a id="step-00-README.md-8ec9a0-diff"></a>

<details><summary>The diff of <code>README.md</code></summary>

````diff
@@ -0,0 +1,14 @@
+# cheesecracker
+
+cheesecracker is the API server of the Cheese App, a class project of AC215. The server answers a few HTTP
+requests. The project shows how to test a server, check its code, and run it in Docker.
+
+## Run it
+
+You need [uv](https://docs.astral.sh/uv/) and [just](https://just.systems/). Then run:
+
+```
+just setup
+```
+
+`just` lists every recipe of the project.
````

</details>

<a id="step-00-justfile-424f0b-diff"></a>

<details><summary>The diff of <code>justfile</code></summary>

```diff
@@ -0,0 +1,12 @@
+# cheesecracker: every task is a recipe here. `just` lists them.
+
+# Each word after a recipe's name reaches its commands whole, as "$1", "$2" and so on; "$@" is all of them
+set positional-arguments
+
+[private]
+default:
+    @just --list --unsorted
+
+# Install Python and every dependency into .venv, exactly as the lockfile says
+setup:
+    uv sync --locked
```

</details>

<a id="step-00-pyproject.toml-5d07e7-diff"></a>

<details><summary>The diff of <code>pyproject.toml</code></summary>

```diff
@@ -0,0 +1,13 @@
+[project]
+name = "cheesecracker"
+version = "0.1.0"
+description = "The Cheese App's API server, with its tests, its checks and its Docker images"
+readme = "README.md"
+requires-python = ">=3.12"
+dependencies = []
+
+[tool.uv]
+# The project is an application, not a library: uv installs its dependencies, and not the project itself
+package = false
+# uv sync and uv run install every dependency group, not only dev
+default-groups = "all"
```

</details>

<a id="step-01"></a>

## 1 A function, and its unit tests

This walk goes through [rahuldave/cheesecracker](https://github.com/rahuldave/cheesecracker) on GitHub. This part starts from the code at `step-00`. [The code before this part](https://github.com/rahuldave/cheesecracker/tree/step-00)

The first code to check is the function `power()`, with its unit tests.

```bash
just setup
```

The server will compute distances, and it needs a function that raises a number to a power. We write the function
first, and tests that call it alone. A unit test calls one function, with no server and no network, so it runs in
milliseconds.

<a id="step-01.1"></a>

### 1.1 power(), the first function of the API

The code of the API is a Python package, `api`, in the folder `src/api-service/`. Its first function, `power()`,
returns its first argument raised to the power of its second.

Apply the patch to [`src/api-service/api/__init__.py`](#step-01.1-src-api-service-api-__init__.py-e2e18a), from the top of the project:

```bash
curl -fsSL https://github.com/rahuldave/cheesecracker/commit/bb1701fd54ba7c6aa9bd971c6651f34439f004d6.diff | git apply --include=src/api-service/api/__init__.py
```

Apply the patch to [`src/api-service/api/utils.py`](#step-01.1-src-api-service-api-utils.py-18f1f8), from the top of the project:

```bash
curl -fsSL https://github.com/rahuldave/cheesecracker/commit/bb1701fd54ba7c6aa9bd971c6651f34439f004d6.diff | git apply --include=src/api-service/api/utils.py
```

Make the folder `src/api-service/api/`, and apply both patches. `__init__.py` has one comment, and makes the folder a
package. In `utils.py`, read the function `power()` after the docstring of the module.

The package is not on the import path of the project yet. So the check runs Python in the folder `src/api-service/`,
where `api` is a folder of its own.

```bash
uv run --directory src/api-service python -c "from api.utils import power; print(power(2, 3))"
```

Python prints `8`, which is 2 to the power of 3.

Check your files against this move. The command prints nothing when they match, and otherwise shows what differs:

```bash
git check bb1701fd54ba
```

If you are stuck, this command puts the files of this move in place of yours. A file that you made yourself stays:

```bash
git fetch --quiet --tags --force origin; git restore --source step-00 --staged -- :/ && git restore --source bb1701fd54ba --staged --worktree -- :/
```

#### What changed

- <a id="step-01.1-src-api-service-api-__init__.py-e2e18a"></a>`src/api-service/api/__init__.py`: one comment, which makes the folder a package. New, 1 line. [The file](https://github.com/rahuldave/cheesecracker/blob/bb1701fd54ba7c6aa9bd971c6651f34439f004d6/src/api-service/api/__init__.py).
- <a id="step-01.1-src-api-service-api-utils.py-18f1f8"></a>`src/api-service/api/utils.py`: the function `power()`. New, 9 lines. [The file](https://github.com/rahuldave/cheesecracker/blob/bb1701fd54ba7c6aa9bd971c6651f34439f004d6/src/api-service/api/utils.py).
  - look at `return base**exponent`

<a id="step-01.1-src-api-service-api-__init__.py-e2e18a-diff"></a>

<details open><summary>The patch to <code>src/api-service/api/__init__.py</code></summary>

```diff
@@ -0,0 +1 @@
+# API Package
```

</details>

<a id="step-01.1-src-api-service-api-utils.py-18f1f8-diff"></a>

<details open><summary>The patch to <code>src/api-service/api/utils.py</code></summary>

```diff
@@ -0,0 +1,9 @@
+"""
+Utility functions for mathematical operations
+Used by the Cheese App API Service
+"""
+
+
+def power(base, exponent):
+    """Calculate base raised to the power of exponent"""
+    return base**exponent
```

</details>

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/bb1701fd54ba7c6aa9bd971c6651f34439f004d6). [The code after it](https://github.com/rahuldave/cheesecracker/tree/bb1701fd54ba7c6aa9bd971c6651f34439f004d6).

<a id="step-01.2"></a>

### 1.2 pytest, in the dependency group test

A test calls `power()` and compares the result with the answer that we expect.
[pytest](https://docs.pytest.org/) is the test runner of most Python projects. It finds each function whose name
starts with `test_`, in the files `test_*.py`, and runs it. A test checks a result with a plain `assert`. pytest
reports each failure with the values that it compared.

The tests need pytest, but the server does not. So pytest goes into a group of
dependencies of its own, `test`.

```bash
uv add --group test pytest
```

The command adds pytest to the group `test` of `pyproject.toml`. It writes the exact versions of pytest and of the
packages that it needs into `uv.lock`.

```bash
uv run pytest --version
```

uv prints the version of pytest, `pytest 9.1.1` on the day of the class.

Check your files against this move. The command prints nothing when they match, and otherwise shows what differs:

```bash
git check fc8c3bce8360
```

If you are stuck, this command puts the files of this move in place of yours. A file that you made yourself stays:

```bash
git restore --source bb1701fd54ba --staged -- :/ && git restore --source fc8c3bce8360 --staged --worktree -- :/
```

#### What changed

- <a id="step-01.2-pyproject.toml-5d07e7"></a>`pyproject.toml`: pytest, in the group `test`. 5 lines added. It comes from `uv add --group test pytest`. [The file](https://github.com/rahuldave/cheesecracker/blob/fc8c3bce83604277393460c4eb243d176d51905c/pyproject.toml).
  - look at `"pytest>=9.1.1",`
- <a id="step-01.2-uv.lock-8356d8"></a>`uv.lock`: the exact version of pytest, and of the packages that it needs. 71 lines added. It comes from `uv add --group test pytest`. [The file](https://github.com/rahuldave/cheesecracker/blob/fc8c3bce83604277393460c4eb243d176d51905c/uv.lock).

<a id="step-01.2-pyproject.toml-5d07e7-diff"></a>

<details><summary>The diff that <code>uv add --group test pytest</code> makes in <code>pyproject.toml</code></summary>

```diff
@@ -11,3 +11,8 @@ dependencies = []
 package = false
 # uv sync and uv run install every dependency group, not only dev
 default-groups = "all"
+
+[dependency-groups]
+test = [
+    "pytest>=9.1.1",
+]
```

</details>

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/fc8c3bce83604277393460c4eb243d176d51905c). [The code after it](https://github.com/rahuldave/cheesecracker/tree/fc8c3bce83604277393460c4eb243d176d51905c).

<a id="step-01.3"></a>

### 1.3 pytest finds the tests, and the package api

pytest looks for tests in every folder of the project, `.venv` too, which is slow. Also, the tests cannot import
`api`, because the folder `src/api-service/` is not on the import path. Two settings of pytest, in `pyproject.toml`, solve both problems.

Set the values that this change sets in [`pyproject.toml`](#step-01.3-pyproject.toml-5d07e7). Run this command from the top of the project. It reads your file with tomlkit, a library for TOML files. It copies each of these tables and values from the project's file after this change, with their comments, and keeps the rest of your file:

```bash
uv run --quiet --no-config --no-project --with tomlkit python - <<'PY'
import copy, pathlib, subprocess, tomlkit
from tomlkit.items import Comment, Table, Whitespace
path = pathlib.Path('pyproject.toml')
doc = tomlkit.parse(path.read_text(encoding='utf-8')) if path.exists() else tomlkit.document()
# The project's file after this change, from your clone
project = tomlkit.parse(subprocess.run(['git', 'show', 'ff82ac1870a7:pyproject.toml'], capture_output=True, text=True,
                                        check=True).stdout)
# Each table and value that the change sets, by its keys: each comes from the project's file with its comments
COPY = [
    ('tool', 'pytest', 'ini_options'),
]

def body(table):   # the items of a table in their order, with its comment lines
    inner = table if isinstance(table, tomlkit.TOMLDocument) else getattr(table, 'value', None)
    return getattr(inner, 'body', [])

def above(table, key):   # the comment lines just above a key
    found = []
    for name, item in body(table):
        if name is not None and name.key == key:
            return found
        found = found + [item] if isinstance(item, Comment) else found if isinstance(item, Whitespace) else []
    return []

for keys in COPY:
    source, target = project, doc
    for key in keys[:-1]:   # make each table on the way
        source = source[key]
        if key not in target:
            target[key] = tomlkit.table(True)
        target = target[key]
    item = copy.deepcopy(source[keys[-1]])
    if isinstance(item, Table) and not item.as_string().endswith('\n\n'):
        item.add(tomlkit.nl())   # a blank line before the table after it
    if keys[-1] not in target:
        for comment in above(source, keys[-1]):
            target.add(copy.deepcopy(comment))
    target[keys[-1]] = item
path.write_text(tomlkit.dumps(doc), encoding='utf-8')
PY
```

Read the patch. It adds the table `[tool.pytest.ini_options]` to `pyproject.toml`.

```bash
uv run pytest
```

pytest collects 0 items, and fails with the code 5, because there are no tests yet. It also warns that it found no
files in `tests/`, which does not exist yet.

Check your files against this move. The command prints nothing when they match, and otherwise shows what differs:

```bash
git check ff82ac1870a7
```

If you are stuck, this command puts the files of this move in place of yours. A file that you made yourself stays:

```bash
git restore --source fc8c3bce8360 --staged -- :/ && git restore --source ff82ac1870a7 --staged --worktree -- :/
```

#### What changed

- <a id="step-01.3-pyproject.toml-5d07e7"></a>`pyproject.toml`: the table `[tool.pytest.ini_options]`, with the folder of the tests and the folder of the package. 5 lines added. [The file](https://github.com/rahuldave/cheesecracker/blob/ff82ac1870a7e1937a6fb4d25ac171249378f299/pyproject.toml).
  - look at `pythonpath = ["src/api-service"]`

<a id="step-01.3-pyproject.toml-5d07e7-diff"></a>

<details open><summary>The patch to <code>pyproject.toml</code></summary>

```diff
@@ -16,3 +16,8 @@ default-groups = "all"
 test = [
     "pytest>=9.1.1",
 ]
+
+[tool.pytest.ini_options]
+# The tests are in tests/, and import the package api from src/api-service/
+testpaths = ["tests"]
+pythonpath = ["src/api-service"]
```

</details>

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/ff82ac1870a7e1937a6fb4d25ac171249378f299). [The code after it](https://github.com/rahuldave/cheesecracker/tree/ff82ac1870a7e1937a6fb4d25ac171249378f299).

<a id="step-01.4"></a>

### 1.4 unit tests of power()

We do not know whether `power()` gives the right answer for every kind of input. Eleven tests try it with integers,
zero, one, negative numbers, floats, the special values of floats, and inputs of the wrong type.

Apply the patch to [`tests/unit/test_utils.py`](#step-01.4-tests-unit-test_utils.py-3d93fe), from the top of the project:

```bash
curl -fsSL https://github.com/rahuldave/cheesecracker/commit/cbc98b162671bce7a7ba82d7d18c7dc6d530b746.diff | git apply --include=tests/unit/test_utils.py
```

Make the folder `tests/unit/`, and apply the patch. Read the class `TestPower`. Each method is one test, and each
`assert` compares a result of `power()` with its answer.

```bash
uv run pytest -q
```

pytest prints `11 passed`. So `power()` gives the answer that each test expects.

Check your files against this move. The command prints nothing when they match, and otherwise shows what differs:

```bash
git check cbc98b162671
```

If you are stuck, this command puts the files of this move in place of yours. A file that you made yourself stays:

```bash
git restore --source ff82ac1870a7 --staged -- :/ && git restore --source cbc98b162671 --staged --worktree -- :/
```

#### What changed

- <a id="step-01.4-tests-unit-test_utils.py-3d93fe"></a>`tests/unit/test_utils.py`: eleven unit tests of `power()`. New, 91 lines. [The file](https://github.com/rahuldave/cheesecracker/blob/cbc98b162671bce7a7ba82d7d18c7dc6d530b746/tests/unit/test_utils.py).
  - look at `assert power(2, 3) == 8`

<a id="step-01.4-tests-unit-test_utils.py-3d93fe-diff"></a>

<details open><summary>The patch to <code>tests/unit/test_utils.py</code></summary>

```diff
@@ -0,0 +1,91 @@
+"""
+Unit tests for utils module
+Tests the power function in isolation
+"""
+
+import pytest
+from api.utils import power
+
+
+class TestPower:
+    """Tests for the power function"""
+
+    def test_power_positive_integers(self):
+        """Test power with positive integers"""
+        assert power(2, 3) == 8
+        assert power(3, 2) == 9
+        assert power(5, 2) == 25
+        assert power(10, 2) == 100
+
+    def test_power_with_zero_exponent(self):
+        """Test power with zero exponent (any number^0 = 1)"""
+        assert power(5, 0) == 1
+        assert power(100, 0) == 1
+        assert power(0, 0) == 1
+
+    def test_power_with_one(self):
+        """Test power with base or exponent of one"""
+        assert power(1, 5) == 1
+        assert power(5, 1) == 5
+        assert power(1, 1) == 1
+
+    def test_power_negative_base(self):
+        """Test power with negative base"""
+        assert power(-2, 2) == 4
+        assert power(-3, 3) == -27
+        assert power(-2, 4) == 16
+        assert power(-5, 2) == 25
+
+    def test_power_negative_exponent(self):
+        """Test power with negative exponent (1/base^exp)"""
+        assert power(2, -1) == pytest.approx(0.5)
+        assert power(2, -2) == pytest.approx(0.25)
+        assert power(10, -1) == pytest.approx(0.1)
+
+    def test_power_with_floats(self):
+        """Test power with floating point numbers"""
+        assert power(2.5, 2) == pytest.approx(6.25)
+        assert power(1.5, 3) == pytest.approx(3.375)
+        assert power(0.5, 2) == pytest.approx(0.25)
+
+    def test_power_large_numbers(self):
+        """Test power with large numbers"""
+        assert power(10, 6) == 1000000
+        assert power(2, 10) == 1024
+
+    def test_power_fractional_exponent(self):
+        """Test power with fractional exponent (roots)"""
+        assert power(4, 0.5) == pytest.approx(2.0)  # square root
+        assert power(8, 1 / 3) == pytest.approx(2.0)  # cube root
+        assert power(16, 0.25) == pytest.approx(2.0)  # fourth root
+
+    def test_power_special_float_values(self):
+        """Test special float values"""
+        import math
+
+        assert math.isnan(power(float("nan"), 2))
+        assert power(float("inf"), 2) == float("inf")
+        assert power(2, float("inf")) == float("inf")
+
+    def test_power_invalid_inputs(self):
+        """Test that invalid inputs raise TypeError"""
+        with pytest.raises(TypeError):
+            power("3", 4)
+        with pytest.raises(TypeError):
+            power(3, "4")
+        with pytest.raises(TypeError):
+            power(None, 4)
+        with pytest.raises(TypeError):
+            power(3, None)
+        with pytest.raises(TypeError):
+            power([3], 4)
+        with pytest.raises(TypeError):
+            power(3, [4])
+        with pytest.raises(TypeError):
+            power({}, 4)
+
+    def test_power_boolean_values(self):
+        """Test with boolean values (True=1, False=0)"""
+        assert power(True, 5) == 1
+        assert power(False, 5) == 0
+        assert power(2, True) == 2
```

</details>

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/cbc98b162671bce7a7ba82d7d18c7dc6d530b746). [The code after it](https://github.com/rahuldave/cheesecracker/tree/cbc98b162671bce7a7ba82d7d18c7dc6d530b746).

<a id="step-01.5"></a>

### 1.5 just test runs the tests

The command `uv run pytest` is long, and every task of the project gets a recipe in the `justfile`, so that one
short command runs it. The recipe `test` runs pytest through uv, and passes on to pytest any words that you type after `just test`.

Apply the patch to [`justfile`](#step-01.5-justfile-424f0b), from the top of the project:

```bash
curl -fsSL https://github.com/rahuldave/cheesecracker/commit/e86e0c3d10bbe9409674f1c65ce4afdc38e7f76f.diff | git apply --include=justfile
```

Read the recipe `test` at the end of the `justfile`.

```bash
just test -q
```

just prints the command of the recipe, and pytest prints `11 passed`.

Check your files against this move. The command prints nothing when they match, and otherwise shows what differs:

```bash
git check e86e0c3d10bb
```

If you are stuck, this command puts the files of this move in place of yours. A file that you made yourself stays:

```bash
git fetch --quiet --tags --force origin; git restore --source cbc98b162671 --staged -- :/ && git restore --source step-01 --staged --worktree -- :/
```

#### What changed

- <a id="step-01.5-justfile-424f0b"></a>`justfile`: the recipe `test`. 4 lines added. [The file](https://github.com/rahuldave/cheesecracker/blob/e86e0c3d10bbe9409674f1c65ce4afdc38e7f76f/justfile).
  - look at `uv run pytest "$@"`

<a id="step-01.5-justfile-424f0b-diff"></a>

<details open><summary>The patch to <code>justfile</code></summary>

```diff
@@ -10,3 +10,7 @@ default:
 # Install Python and every dependency into .venv, exactly as the lockfile says
 setup:
     uv sync --locked
+
+# Run the tests. Extra arguments go to pytest: just test tests/unit
+test *args:
+    uv run pytest "$@"
```

</details>

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/e86e0c3d10bbe9409674f1c65ce4afdc38e7f76f).

This part ends at the code at `step-01`. [The code after this part](https://github.com/rahuldave/cheesecracker/tree/step-01)

<a id="step-02"></a>

## 2 The server

This walk goes through [rahuldave/cheesecracker](https://github.com/rahuldave/cheesecracker) on GitHub. This part starts from the code at `step-01`. [The code before this part](https://github.com/rahuldave/cheesecracker/tree/step-01)

The server answers two requests, `/` and `/health`.

```bash
just setup
```

The Cheese App talks to its server over HTTP. FastAPI turns Python functions into the answers of HTTP requests, and
uvicorn is the server that runs a FastAPI app. We write an app that answers two requests, and run it on this machine.

<a id="step-02.1"></a>

### 2.1 FastAPI, and uvicorn to serve it

[FastAPI](https://fastapi.tiangolo.com/) is a library for web APIs. A path is the part of an address after the name of the
server, such as `/health`. A Python function answers the requests to one path when a line such as `@app.get("/health")` stands just above it. FastAPI uses the type written next to each argument, such
as `float`, to check the values of each request. It also writes the pages of the API's docs.

[uvicorn](https://uvicorn.dev/) is the server. It waits for requests on a numbered port of the machine, such as 9000,
and hands each request to the app.

The server needs FastAPI and uvicorn to run, so they are dependencies of the project itself, and not of a group.

```bash
uv add fastapi uvicorn
```

The command adds both packages to `dependencies` in `pyproject.toml`, and writes their versions into `uv.lock`.

```bash
uv run uvicorn --version
```

uvicorn prints its version, and the version of Python that runs it.

Check your files against this move. The command prints nothing when they match, and otherwise shows what differs:

```bash
git check 18dd9dfb1c95
```

If you are stuck, this command puts the files of this move in place of yours. A file that you made yourself stays:

```bash
git fetch --quiet --tags --force origin; git restore --source step-01 --staged -- :/ && git restore --source 18dd9dfb1c95 --staged --worktree -- :/
```

#### What changed

- <a id="step-02.1-pyproject.toml-5d07e7"></a>`pyproject.toml`: FastAPI and uvicorn, the dependencies of the server. 4 lines added, 1 removed. It comes from `uv add fastapi uvicorn`. [The file](https://github.com/rahuldave/cheesecracker/blob/18dd9dfb1c9597e50a8b404a6cb6379c1c149320/pyproject.toml).
  - look at `"fastapi>=0.143.2",`
- <a id="step-02.1-uv.lock-8356d8"></a>`uv.lock`: the exact versions of FastAPI, uvicorn, and the packages that they need. 263 lines added. It comes from `uv add fastapi uvicorn`. [The file](https://github.com/rahuldave/cheesecracker/blob/18dd9dfb1c9597e50a8b404a6cb6379c1c149320/uv.lock).

<a id="step-02.1-pyproject.toml-5d07e7-diff"></a>

<details><summary>The diff that <code>uv add fastapi uvicorn</code> makes in <code>pyproject.toml</code></summary>

```diff
@@ -4,7 +4,10 @@ version = "0.1.0"
 description = "The Cheese App's API server, with its tests, its checks and its Docker images"
 readme = "README.md"
 requires-python = ">=3.12"
-dependencies = []
+dependencies = [
+    "fastapi>=0.143.2",
+    "uvicorn>=0.54.0",
+]
 
 [tool.uv]
 # The project is an application, not a library: uv installs its dependencies, and not the project itself
```

</details>

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/18dd9dfb1c9597e50a8b404a6cb6379c1c149320). [The code after it](https://github.com/rahuldave/cheesecracker/tree/18dd9dfb1c9597e50a8b404a6cb6379c1c149320).

<a id="step-02.2"></a>

### 2.2 the API server, with / and /health

The app needs a first answer, and a way to tell that it runs. `service.py` makes the FastAPI app, and answers two
requests. The request `/` gets a welcome, and `/health` gets the status of the server.

Apply the patch to [`src/api-service/api/service.py`](#step-02.2-src-api-service-api-service.py-1e1e71), from the top of the project:

```bash
curl -fsSL https://github.com/rahuldave/cheesecracker/commit/acee14f7dff43d6c5b76234b3a016985d74c60d2.diff | git apply --include=src/api-service/api/service.py
```

Apply the patch, and read the file from the top. It makes `app`, and adds the CORS middleware, which lets a web page
from any address call the server. Then it defines the two routes after the comment `# Routes`.

```bash
uv run --directory src/api-service python -c "from api.service import app; print([r.path for r in app.routes])"
```

Python prints the routes of the app. The last two are `/` and `/health`. FastAPI adds the first four, for its
documentation pages.

Check your files against this move. The command prints nothing when they match, and otherwise shows what differs:

```bash
git check acee14f7dff4
```

If you are stuck, this command puts the files of this move in place of yours. A file that you made yourself stays:

```bash
git restore --source 18dd9dfb1c95 --staged -- :/ && git restore --source acee14f7dff4 --staged --worktree -- :/
```

#### What changed

- <a id="step-02.2-src-api-service-api-service.py-1e1e71"></a>`src/api-service/api/service.py`: the FastAPI app, its CORS middleware, and the routes `/` and `/health`. New, 29 lines. [The file](https://github.com/rahuldave/cheesecracker/blob/acee14f7dff43d6c5b76234b3a016985d74c60d2/src/api-service/api/service.py).
  - look at `return {"status": "healthy"}`

<a id="step-02.2-src-api-service-api-service.py-1e1e71-diff"></a>

<details open><summary>The patch to <code>src/api-service/api/service.py</code></summary>

```diff
@@ -0,0 +1,29 @@
+"""
+Cheese App API Service
+"""
+
+from fastapi import FastAPI
+from starlette.middleware.cors import CORSMiddleware
+
+# Setup FastAPI app
+app = FastAPI(title="Cheese API Server", description="API Server for Cheese App", version="v1")
+
+# Enable CORSMiddleware
+app.add_middleware(
+    CORSMiddleware,
+    allow_credentials=False,
+    allow_origins=["*"],
+    allow_methods=["*"],
+    allow_headers=["*"],
+)
+
+
+# Routes
+@app.get("/")
+async def get_index():
+    return {"message": "Welcome to the Cheese App CI/CD Tutorial!"}
+
+
+@app.get("/health")
+async def health_check():
+    return {"status": "healthy"}
```

</details>

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/acee14f7dff43d6c5b76234b3a016985d74c60d2). [The code after it](https://github.com/rahuldave/cheesecracker/tree/acee14f7dff43d6c5b76234b3a016985d74c60d2).

<a id="step-02.3"></a>

### 2.3 just serve and just stop, the server on this machine

The app does nothing until a server runs it. The recipe `serve` starts uvicorn at port 9000, in the background, and
waits until the server answers. The recipe `stop` stops it. A server is a fixture of the commands that need it. We start
it before them, and stop it after them.

Apply the patch to [`justfile`](#step-02.3-justfile-424f0b), from the top of the project:

```bash
curl -fsSL https://github.com/rahuldave/cheesecracker/commit/69c3bf65c78cf8e96bf8f9ea1dde29279db1351a.diff | git apply --include=justfile
```

Apply the patch to [`.gitignore`](#step-02.3-.gitignore-a5cc29), from the top of the project:

```bash
curl -fsSL https://github.com/rahuldave/cheesecracker/commit/69c3bf65c78cf8e96bf8f9ea1dde29279db1351a.diff | git apply --include=.gitignore
```

Read the recipes `serve` and `stop` at the end of the `justfile`. Then read the two new lines of `.gitignore`.

```bash
just serve
```

The recipe prints `serve: http://localhost:9000, its log in .serve.log`. Ask the server for its health.

```bash
curl -s localhost:9000/health
```

[curl](https://curl.se/) sends an HTTP request from the terminal, and prints the answer. macOS and most Linux systems
have it. The server answers `{"status":"healthy"}`. Open `http://localhost:9000/docs` in a browser to see the pages that
FastAPI makes for each route.

```bash
just stop
```

The recipe prints `stop: stopped`. Ask for the health again.

```bash
curl -s localhost:9000/health
```

`curl` prints nothing, and fails with the code 7, because no server answers.

Check your files against this move. The command prints nothing when they match, and otherwise shows what differs:

```bash
git check 69c3bf65c78c
```

If you are stuck, this command puts the files of this move in place of yours. A file that you made yourself stays:

```bash
git fetch --quiet --tags --force origin; git restore --source acee14f7dff4 --staged -- :/ && git restore --source step-02 --staged --worktree -- :/
```

#### What changed

- <a id="step-02.3-.gitignore-a5cc29"></a>`.gitignore`: the log of the server and the number of its process. 4 lines added. [The file](https://github.com/rahuldave/cheesecracker/blob/69c3bf65c78cf8e96bf8f9ea1dde29279db1351a/.gitignore).
  - look at `.serve.pid`
- <a id="step-02.3-justfile-424f0b"></a>`justfile`: the recipes `serve` and `stop`. 26 lines added. [The file](https://github.com/rahuldave/cheesecracker/blob/69c3bf65c78cf8e96bf8f9ea1dde29279db1351a/justfile).
  - look at `uv run uvicorn api.service:app --app-dir src/api-service --host 0.0.0.0 --port 9000 --reload > .serve.log 2>&1 &`

<a id="step-02.3-.gitignore-a5cc29-diff"></a>

<details open><summary>The patch to <code>.gitignore</code></summary>

```diff
@@ -9,6 +9,10 @@ __pycache__/
 htmlcov/
 .ruff_cache/
 
+# What just serve writes: the log of the server, and its process id
+.serve.log
+.serve.pid
+
 # Secrets: never commit them
 secrets/
 .env
```

</details>

<a id="step-02.3-justfile-424f0b-diff"></a>

<details open><summary>The patch to <code>justfile</code></summary>

```diff
@@ -14,3 +14,29 @@ setup:
 # Run the tests. Extra arguments go to pytest: just test tests/unit
 test *args:
     uv run pytest "$@"
+
+# Start the server on this machine, in the background, at http://localhost:9000. It restarts when a file of the code changes. just stop stops it
+serve:
+    #!/usr/bin/env sh
+    if curl -sf localhost:9000/health > /dev/null; then echo "serve: a server already answers at http://localhost:9000" >&2; exit 1; fi
+    uv run uvicorn api.service:app --app-dir src/api-service --host 0.0.0.0 --port 9000 --reload > .serve.log 2>&1 &
+    echo $! > .serve.pid
+    for i in $(seq 1 30); do
+        if curl -sf localhost:9000/health > /dev/null; then echo "serve: http://localhost:9000, its log in .serve.log"; exit 0; fi
+        sleep 1
+    done
+    echo "serve: no answer after 30 seconds; read .serve.log" >&2
+    exit 1
+
+# Stop the server of just serve
+stop:
+    #!/usr/bin/env sh
+    if [ ! -f .serve.pid ]; then echo "stop: no server of just serve runs here" >&2; exit 1; fi
+    kill "$(cat .serve.pid)" && rm .serve.pid
+    # Wait until the server no longer answers, so that the port is free for the next one
+    for i in $(seq 1 30); do
+        if ! curl -s localhost:9000/health > /dev/null; then echo "stop: stopped"; exit 0; fi
+        sleep 1
+    done
+    echo "stop: the server still answers after 30 seconds" >&2
+    exit 1
```

</details>

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/69c3bf65c78cf8e96bf8f9ea1dde29279db1351a).

This part ends at the code at `step-02`. [The code after this part](https://github.com/rahuldave/cheesecracker/tree/step-02)

<a id="step-03"></a>

## 3 The distance endpoint, and integration tests

This walk goes through [rahuldave/cheesecracker](https://github.com/rahuldave/cheesecracker) on GitHub. This part starts from the code at `step-02`. [The code before this part](https://github.com/rahuldave/cheesecracker/tree/step-02)

The server gets a new request, `/euclidean_distance/`, which works out how far a point is from the origin. New tests
send that request to the app, and check the answer.

```bash
just setup
```

The app needs an endpoint that does some work, and tests that call it. The endpoint computes the distance of a point
from the origin with `power()`. An integration test sends requests to the app and checks the answers, with no server
and no network.

<a id="step-03.1"></a>

### 3.1 /euclidean_distance/, the distance of a point from the origin

The Cheese App asks the server for the distance of a point `(x, y)` from the origin `(0, 0)`. That distance is the
square root of x squared plus y squared.

Apply the patch to [`src/api-service/api/service.py`](#step-03.1-src-api-service-api-service.py-1e1e71), from the top of the project:

```bash
curl -fsSL https://github.com/rahuldave/cheesecracker/commit/e576e7bb9e550b1a572a764b15f8d0bfde229d97.diff | git apply --include=src/api-service/api/service.py
```

Read the two imports after `CORSMiddleware`, and the new function at the end of the file.

Start the server, and ask it for the distance of the point (3, 4). The server runs to the end of this part, and
restarts when a file of the code changes, because of `--reload`.

```bash
just serve
curl -s "localhost:9000/euclidean_distance/?x=3&y=4"
```

The server answers `{"x":3.0,"y":4.0,"distance":5.0}`.

Check your files against this move. The command prints nothing when they match, and otherwise shows what differs:

```bash
git check e576e7bb9e55
```

If you are stuck, this command puts the files of this move in place of yours. A file that you made yourself stays:

```bash
git fetch --quiet --tags --force origin; git restore --source step-02 --staged -- :/ && git restore --source e576e7bb9e55 --staged --worktree -- :/
```

#### What changed

- <a id="step-03.1-src-api-service-api-service.py-1e1e71"></a>`src/api-service/api/service.py`: the imports of `math` and `power()`, and the endpoint `/euclidean_distance/`. 12 lines added. [The file](https://github.com/rahuldave/cheesecracker/blob/e576e7bb9e550b1a572a764b15f8d0bfde229d97/src/api-service/api/service.py).
  - look at `z = power(x, 2) + power(y, 2)`

<a id="step-03.1-src-api-service-api-service.py-1e1e71-diff"></a>

<details open><summary>The patch to <code>src/api-service/api/service.py</code></summary>

```diff
@@ -4,6 +4,8 @@ Cheese App API Service
 
 from fastapi import FastAPI
 from starlette.middleware.cors import CORSMiddleware
+import math
+from api.utils import power
 
 # Setup FastAPI app
 app = FastAPI(title="Cheese API Server", description="API Server for Cheese App", version="v1")
@@ -27,3 +29,13 @@ async def get_index():
 @app.get("/health")
 async def health_check():
     return {"status": "healthy"}
+
+
+@app.get("/euclidean_distance/")
+async def calculate_euclidean_distance(x: float = 1, y: float = 2):
+    """Calculate the Euclidean distance from origin: sqrt(x^2 + y^2)
+    Returns the distance of a point (x, y) from the origin (0, 0)
+    """
+    z = power(x, 2) + power(y, 2)
+    result = math.sqrt(z)
+    return {"x": x, "y": y, "distance": result}
```

</details>

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/e576e7bb9e550b1a572a764b15f8d0bfde229d97). [The code after it](https://github.com/rahuldave/cheesecracker/tree/e576e7bb9e550b1a572a764b15f8d0bfde229d97).

<a id="step-03.2"></a>

### 3.2 httpx2 and pytest-asyncio, for tests of the app

FastAPI's [`TestClient`](https://fastapi.tiangolo.com/tutorial/testing/) sends requests to the app in the same
process, with no server. It needs [httpx2](https://github.com/pydantic/httpx2), a library that sends HTTP requests.
One test of the app is a coroutine, an `async` function. pytest cannot run an `async` function by itself, so
[pytest-asyncio](https://pytest-asyncio.readthedocs.io/) runs it.

```bash
uv add --group test httpx2 pytest-asyncio
```

The command adds both packages to the group `test`, and writes their versions into `uv.lock`.

```bash
just test -q
```

pytest still prints `11 passed`. The new packages change no test.

Check your files against this move. The command prints nothing when they match, and otherwise shows what differs:

```bash
git check 3c75ffb1a089
```

If you are stuck, this command puts the files of this move in place of yours. A file that you made yourself stays:

```bash
git restore --source e576e7bb9e55 --staged -- :/ && git restore --source 3c75ffb1a089 --staged --worktree -- :/
```

#### What changed

- <a id="step-03.2-pyproject.toml-5d07e7"></a>`pyproject.toml`: httpx2 and pytest-asyncio, in the group `test`. 2 lines added. It comes from `uv add --group test httpx2 pytest-asyncio`. [The file](https://github.com/rahuldave/cheesecracker/blob/3c75ffb1a089431a2455a69e9bcb3ec0302224f0/pyproject.toml).
  - look at `"httpx2>=2.13.1",`
- <a id="step-03.2-uv.lock-8356d8"></a>`uv.lock`: the exact versions of httpx2, pytest-asyncio, and the packages that they need. 68 lines added, 1 removed. It comes from `uv add --group test httpx2 pytest-asyncio`. [The file](https://github.com/rahuldave/cheesecracker/blob/3c75ffb1a089431a2455a69e9bcb3ec0302224f0/uv.lock).

<a id="step-03.2-pyproject.toml-5d07e7-diff"></a>

<details><summary>The diff that <code>uv add --group test httpx2 pytest-asyncio</code> makes in <code>pyproject.toml</code></summary>

```diff
@@ -17,7 +17,9 @@ default-groups = "all"
 
 [dependency-groups]
 test = [
+    "httpx2>=2.13.1",
     "pytest>=9.1.1",
+    "pytest-asyncio>=1.4.0",
 ]
 
 [tool.pytest.ini_options]
```

</details>

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/3c75ffb1a089431a2455a69e9bcb3ec0302224f0). [The code after it](https://github.com/rahuldave/cheesecracker/tree/3c75ffb1a089431a2455a69e9bcb3ec0302224f0).

<a id="step-03.3"></a>

### 3.3 integration tests of the API

A unit test of `power()` cannot tell whether the app routes a request to the right function, or reads its
parameters, or answers in JSON. Ten integration tests send requests to the app through `TestClient`, and check each
answer.

Apply the patch to [`tests/integration/test_api.py`](#step-03.3-tests-integration-test_api.py-596674), from the top of the project:

```bash
curl -fsSL https://github.com/rahuldave/cheesecracker/commit/5862780b44d2f4ec87088c971dec57f23c11139e.diff | git apply --include=tests/integration/test_api.py
```

Make the folder `tests/integration/`, and apply the patch. Read the line `client = TestClient(app)`, then the classes
`TestAPIEndpoints` and `TestCORS`.

```bash
just test -q
```

pytest prints `21 passed`, the eleven unit tests and the ten integration tests. The tests needed no server, so we
stop the one of `just serve`.

```bash
just stop
```

Check your files against this move. The command prints nothing when they match, and otherwise shows what differs:

```bash
git check 5862780b44d2
```

If you are stuck, this command puts the files of this move in place of yours. A file that you made yourself stays:

```bash
git fetch --quiet --tags --force origin; git restore --source 3c75ffb1a089 --staged -- :/ && git restore --source step-03 --staged --worktree -- :/
```

#### What changed

- <a id="step-03.3-tests-integration-test_api.py-596674"></a>`tests/integration/test_api.py`: ten integration tests, through FastAPI's `TestClient`. New, 89 lines. [The file](https://github.com/rahuldave/cheesecracker/blob/5862780b44d2f4ec87088c971dec57f23c11139e/tests/integration/test_api.py).
  - look at `response = client.get("/euclidean_distance/?x=3&y=4")`

<a id="step-03.3-tests-integration-test_api.py-596674-diff"></a>

<details open><summary>The patch to <code>tests/integration/test_api.py</code></summary>

```diff
@@ -0,0 +1,89 @@
+"""
+Integration tests for the Cheese App API
+Tests the full API endpoints with FastAPI TestClient
+Based on L18, Slide 11-12 - Integration tests
+"""
+
+import pytest
+from fastapi.testclient import TestClient
+from api.service import app
+
+client = TestClient(app)
+
+
+class TestAPIEndpoints:
+    """Integration tests for API endpoints"""
+
+    def test_root_endpoint(self):
+        """Test the root endpoint returns welcome message"""
+        response = client.get("/")
+        assert response.status_code == 200
+        data = response.json()
+        assert "message" in data
+        assert "Cheese App" in data["message"]
+
+    def test_root_returns_json(self):
+        """Test that root returns JSON content type"""
+        response = client.get("/")
+        assert response.status_code == 200
+        assert "application/json" in response.headers["content-type"]
+
+    def test_euclidean_distance_endpoint(self):
+        """Test the euclidean_distance endpoint"""
+        response = client.get("/euclidean_distance/?x=3&y=4")
+        assert response.status_code == 200
+        data = response.json()
+        assert data["x"] == 3
+        assert data["y"] == 4
+        assert data["distance"] == 5.0
+
+    def test_euclidean_distance_default_parameters(self):
+        """Test euclidean_distance with default parameters"""
+        response = client.get("/euclidean_distance/")
+        assert response.status_code == 200
+        data = response.json()
+        assert "distance" in data
+
+    def test_euclidean_distance_with_floats(self):
+        """Test euclidean_distance with floating point numbers"""
+        response = client.get("/euclidean_distance/?x=1.5&y=2.0")
+        assert response.status_code == 200
+        data = response.json()
+        assert data["x"] == 1.5
+        assert data["y"] == 2.0
+        assert isinstance(data["distance"], float)
+
+    def test_invalid_route_returns_404(self):
+        """Test that invalid routes return 404"""
+        response = client.get("/this-route-does-not-exist")
+        assert response.status_code == 404
+
+    def test_method_not_allowed(self):
+        """Test that POST to GET-only endpoint returns 405"""
+        response = client.post("/")
+        assert response.status_code == 405
+
+
+class TestCORS:
+    """Tests for CORS configuration"""
+
+    def test_cors_enabled(self):
+        """Test that CORS headers are present"""
+        response = client.get("/", headers={"Origin": "http://localhost:3000"})
+        assert response.status_code == 200
+        assert "access-control-allow-origin" in response.headers
+
+    def test_cors_allows_all_origins(self):
+        """Test that CORS allows all origins"""
+        response = client.get("/", headers={"Origin": "http://example.com"})
+        assert response.status_code == 200
+        assert response.headers["access-control-allow-origin"] == "*"
+
+
+@pytest.mark.asyncio
+async def test_api_health_check():
+    """Test that the API is healthy and responding"""
+    response = client.get("/")
+    assert response.status_code == 200
+    # API should respond quickly
+    assert response.elapsed.total_seconds() < 1 if hasattr(response, "elapsed") else True
```

</details>

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/5862780b44d2f4ec87088c971dec57f23c11139e).

This part ends at the code at `step-03`. [The code after this part](https://github.com/rahuldave/cheesecracker/tree/step-03)

<a id="step-05.1"></a>

## 5.1 A change in a hurry

This walk goes through [rahuldave/cheesecracker](https://github.com/rahuldave/cheesecracker) on GitHub. This part starts from the code at `step-04`. [The code before this part](https://github.com/rahuldave/cheesecracker/tree/step-04)

This part does not start where the part before it ends, because the walk skips some changes. This command puts the files that this part starts from in place of yours. Run it, and then go on:

```bash
git fetch --quiet --tags --force origin; git restore --source step-03 --staged -- :/ && git restore --source step-04 --staged --worktree -- :/
```

```bash
just setup
```

The whole tutorial then adds pytest-cov, a tool that shows which lines of the code the tests run, and a recipe,
`just cov`, that runs it. The code of the server does not change.

Then someone changes the code in a hurry. It still works, and it reads worse.

### 5.1 a change in a hurry: the code still works, and reads worse

A change made in a hurry leaves code that works and reads badly. The change adds an import that nothing uses, and takes
out spaces and blank lines. It also makes one line 187 characters long, and adds a function that nothing calls.

Apply the patch to [`src/api-service/api/service.py`](#step-05.1-src-api-service-api-service.py-1e1e71), from the top of the project:

```bash
curl -fsSL https://github.com/rahuldave/cheesecracker/commit/f77650600c1176caa9aaee86e6cddfccc43fc69c.diff | git apply --include=src/api-service/api/service.py
```

Apply the patch to [`src/api-service/api/utils.py`](#step-05.1-src-api-service-api-utils.py-18f1f8), from the top of the project:

```bash
curl -fsSL https://github.com/rahuldave/cheesecracker/commit/f77650600c1176caa9aaee86e6cddfccc43fc69c.diff | git apply --include=src/api-service/api/utils.py
```

Apply both patches. In `service.py`, read the line `import os`, the line `z=power(x,2)+power(y,2)`, and the `return` at
the end. In `utils.py`, read `def power(base,exponent):` and the new function `unused_function()`.

```bash
just test -q
```

pytest still prints `21 passed`. The tests check what the code does, not how it reads.

Check your files against this move. The command prints nothing when they match, and otherwise shows what differs:

```bash
git check f77650600c11
```

If you are stuck, this command puts the files of this move in place of yours. A file that you made yourself stays:

```bash
git fetch --quiet --tags --force origin; git restore --source step-04 --staged -- :/ && git restore --source f77650600c11 --staged --worktree -- :/
```

#### What changed

- <a id="step-05.1-src-api-service-api-service.py-1e1e71"></a>`src/api-service/api/service.py`: an import that nothing uses, missing spaces and blank lines, and a long line. 3 lines added, 5 removed. [The file](https://github.com/rahuldave/cheesecracker/blob/f77650600c1176caa9aaee86e6cddfccc43fc69c/src/api-service/api/service.py).
  - look at `z=power(x,2)+power(y,2)`
- <a id="step-05.1-src-api-service-api-utils.py-18f1f8"></a>`src/api-service/api/utils.py`: a missing space, extra blank lines, and a function that nothing calls. 6 lines added, 1 removed. [The file](https://github.com/rahuldave/cheesecracker/blob/f77650600c1176caa9aaee86e6cddfccc43fc69c/src/api-service/api/utils.py).
  - look at `def unused_function():`

<a id="step-05.1-src-api-service-api-service.py-1e1e71-diff"></a>

<details open><summary>The patch to <code>src/api-service/api/service.py</code></summary>

```diff
@@ -5,6 +5,7 @@ Cheese App API Service
 from fastapi import FastAPI
 from starlette.middleware.cors import CORSMiddleware
 import math
+import os
 from api.utils import power
 
 # Setup FastAPI app
@@ -19,13 +20,10 @@ app.add_middleware(
     allow_headers=["*"],
 )
 
-
 # Routes
 @app.get("/")
 async def get_index():
     return {"message": "Welcome to the Cheese App CI/CD Tutorial!"}
-
-
 @app.get("/health")
 async def health_check():
     return {"status": "healthy"}
@@ -36,6 +34,6 @@ async def calculate_euclidean_distance(x: float = 1, y: float = 2):
     """Calculate the Euclidean distance from origin: sqrt(x^2 + y^2)
     Returns the distance of a point (x, y) from the origin (0, 0)
     """
-    z = power(x, 2) + power(y, 2)
+    z=power(x,2)+power(y,2)
     result = math.sqrt(z)
-    return {"x": x, "y": y, "distance": result}
+    return {"x": x, "y": y, "distance": result, "message": "This is a very long line that exceeds 120 characters and Black will complain about it so we need to fix this formatting issue"}
```

</details>

<a id="step-05.1-src-api-service-api-utils.py-18f1f8-diff"></a>

<details open><summary>The patch to <code>src/api-service/api/utils.py</code></summary>

```diff
@@ -4,6 +4,11 @@ Used by the Cheese App API Service
 """
 
 
-def power(base, exponent):
+def power(base,exponent):
     """Calculate base raised to the power of exponent"""
     return base**exponent
+
+
+
+def unused_function():
+    pass
```

</details>

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/f77650600c1176caa9aaee86e6cddfccc43fc69c).

This part ends at the code at `f77650600c11`, partway through `step-05`. [The code after this part](https://github.com/rahuldave/cheesecracker/tree/f77650600c1176caa9aaee86e6cddfccc43fc69c)

<a id="step-05.2-to-step-05.4"></a>

## 5.2 to 5.4 ruff, the formatter and the linter

This walk goes through [rahuldave/cheesecracker](https://github.com/rahuldave/cheesecracker) on GitHub. This part starts from the code at `f77650600c11`, partway through `step-05`. [The code before this part](https://github.com/rahuldave/cheesecracker/tree/f77650600c1176caa9aaee86e6cddfccc43fc69c)

```bash
just setup
```

ruff goes into a dependency group of its own, `lint`. Its settings allow lines of up to 120 characters, and check the
rules that flake8, an older linter, checks. Two recipes run it. `just fmt` runs the formatter, and `just lint` runs the
linter.

<a id="step-05.2"></a>

### 5.2 ruff, the formatter and the linter, in the group lint

[ruff](https://docs.astral.sh/ruff/) is a formatter and a linter in one tool, written in Rust by Astral, the makers
of uv. `ruff format` writes the code in the layout of black, the formatter of most Python projects. `ruff check`
checks the rules of flake8 and of many other linters, and can fix some of them. It takes the place of both black and
flake8, and runs much faster.

ruff checks the code, and the server does not need it to run. So ruff goes into a group of its own, `lint`.

```bash
uv add --group lint ruff
```

```bash
uv run ruff --version
```

uv prints the version of ruff, `ruff 0.17.1` on the day of the class.

Check your files against this move. The command prints nothing when they match, and otherwise shows what differs:

```bash
git check 9662828462ba
```

If you are stuck, this command puts the files of this move in place of yours. A file that you made yourself stays:

```bash
git restore --source f77650600c11 --staged -- :/ && git restore --source 9662828462ba --staged --worktree -- :/
```

#### What changed

- <a id="step-05.2-pyproject.toml-5d07e7"></a>`pyproject.toml`: ruff, in the group `lint`. 3 lines added. It comes from `uv add --group lint ruff`. [The file](https://github.com/rahuldave/cheesecracker/blob/9662828462ba2a21ca04a79c412e83ac070767f6/pyproject.toml).
  - look at `"ruff>=0.17.1",`
- <a id="step-05.2-uv.lock-8356d8"></a>`uv.lock`: the exact version of ruff. 29 lines added. It comes from `uv add --group lint ruff`. [The file](https://github.com/rahuldave/cheesecracker/blob/9662828462ba2a21ca04a79c412e83ac070767f6/uv.lock).

<a id="step-05.2-pyproject.toml-5d07e7-diff"></a>

<details><summary>The diff that <code>uv add --group lint ruff</code> makes in <code>pyproject.toml</code></summary>

```diff
@@ -16,6 +16,9 @@ package = false
 default-groups = "all"
 
 [dependency-groups]
+lint = [
+    "ruff>=0.17.1",
+]
 test = [
     "httpx2>=2.13.1",
     "pytest>=9.1.1",
```

</details>

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/9662828462ba2a21ca04a79c412e83ac070767f6). [The code after it](https://github.com/rahuldave/cheesecracker/tree/9662828462ba2a21ca04a79c412e83ac070767f6).

<a id="step-05.3"></a>

### 5.3 ruff's settings: lines of 120 characters, and flake8's rules

By default, ruff allows lines of up to 88 characters, and this project allows 120. Its default rules also leave out
some rules of flake8. Two tables in `pyproject.toml` set both.

Set the values that this change sets in [`pyproject.toml`](#step-05.3-pyproject.toml-5d07e7). Run this command from the top of the project. It reads your file with tomlkit, a library for TOML files. It copies each of these tables and values from the project's file after this change, with their comments, and keeps the rest of your file:

```bash
uv run --quiet --no-config --no-project --with tomlkit python - <<'PY'
import copy, pathlib, subprocess, tomlkit
from tomlkit.items import Comment, Table, Whitespace
path = pathlib.Path('pyproject.toml')
doc = tomlkit.parse(path.read_text(encoding='utf-8')) if path.exists() else tomlkit.document()
# The project's file after this change, from your clone
project = tomlkit.parse(subprocess.run(['git', 'show', '7c1670fd52a0:pyproject.toml'], capture_output=True, text=True,
                                        check=True).stdout)
# Each table and value that the change sets, by its keys: each comes from the project's file with its comments
COPY = [
    ('tool', 'ruff'),
]

def body(table):   # the items of a table in their order, with its comment lines
    inner = table if isinstance(table, tomlkit.TOMLDocument) else getattr(table, 'value', None)
    return getattr(inner, 'body', [])

def above(table, key):   # the comment lines just above a key
    found = []
    for name, item in body(table):
        if name is not None and name.key == key:
            return found
        found = found + [item] if isinstance(item, Comment) else found if isinstance(item, Whitespace) else []
    return []

for keys in COPY:
    source, target = project, doc
    for key in keys[:-1]:   # make each table on the way
        source = source[key]
        if key not in target:
            target[key] = tomlkit.table(True)
        target = target[key]
    item = copy.deepcopy(source[keys[-1]])
    if isinstance(item, Table) and not item.as_string().endswith('\n\n'):
        item.add(tomlkit.nl())   # a blank line before the table after it
    if keys[-1] not in target:
        for comment in above(source, keys[-1]):
            target.add(copy.deepcopy(comment))
    target[keys[-1]] = item
path.write_text(tomlkit.dumps(doc), encoding='utf-8')
PY
```

Read the tables `[tool.ruff]` and `[tool.ruff.lint]` of `pyproject.toml`.

```bash
uv run ruff check
```

ruff finds 2 errors, and fails. `F401` is the import of `os`, which nothing uses. `E501` is the line of 187
characters. ruff says that it can fix the first one.

```bash
uv run ruff format --check
```

ruff says `2 files would be reformatted`, and fails. `--check` changes no file.

Check your files against this move. The command prints nothing when they match, and otherwise shows what differs:

```bash
git check 7c1670fd52a0
```

If you are stuck, this command puts the files of this move in place of yours. A file that you made yourself stays:

```bash
git restore --source 9662828462ba --staged -- :/ && git restore --source 7c1670fd52a0 --staged --worktree -- :/
```

#### What changed

- <a id="step-05.3-pyproject.toml-5d07e7"></a>`pyproject.toml`: the settings of ruff, the length of a line and the rules of the linter. 8 lines added. [The file](https://github.com/rahuldave/cheesecracker/blob/7c1670fd52a0f62f414f31d415d10f2efb5a8405/pyproject.toml).
  - look at `select = ["E", "W", "F"]`

<a id="step-05.3-pyproject.toml-5d07e7-diff"></a>

<details open><summary>The patch to <code>pyproject.toml</code></summary>

```diff
@@ -30,3 +30,11 @@ test = [
 # The tests are in tests/, and import the package api from src/api-service/
 testpaths = ["tests"]
 pythonpath = ["src/api-service"]
+
+[tool.ruff]
+# The longest line, for the formatter and the linter
+line-length = 120
+
+[tool.ruff.lint]
+# The rules that flake8 checks: pycodestyle's errors (E) and warnings (W), and pyflakes (F)
+select = ["E", "W", "F"]
```

</details>

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/7c1670fd52a0f62f414f31d415d10f2efb5a8405). [The code after it](https://github.com/rahuldave/cheesecracker/tree/7c1670fd52a0f62f414f31d415d10f2efb5a8405).

<a id="step-05.4"></a>

### 5.4 just fmt and just lint run ruff

Two recipes give the formatter and the linter short names. Each passes on to ruff any words that you type after it.

Apply the patch to [`justfile`](#step-05.4-justfile-424f0b), from the top of the project:

```bash
curl -fsSL https://github.com/rahuldave/cheesecracker/commit/b947593e84b71486902b3ff1c7f9e1458b70e3f7.diff | git apply --include=justfile
```

Read the recipes `fmt` and `lint` at the end of the `justfile`.

```bash
just fmt --check
```

The recipe fails as `ruff format --check` did, with `2 files would be reformatted`.

Check your files against this move. The command prints nothing when they match, and otherwise shows what differs:

```bash
git check b947593e84b7
```

If you are stuck, this command puts the files of this move in place of yours. A file that you made yourself stays:

```bash
git restore --source 7c1670fd52a0 --staged -- :/ && git restore --source b947593e84b7 --staged --worktree -- :/
```

#### What changed

- <a id="step-05.4-justfile-424f0b"></a>`justfile`: the recipes `fmt` and `lint`. 8 lines added. [The file](https://github.com/rahuldave/cheesecracker/blob/b947593e84b71486902b3ff1c7f9e1458b70e3f7/justfile).
  - look at `uv run ruff check "$@"`

<a id="step-05.4-justfile-424f0b-diff"></a>

<details open><summary>The patch to <code>justfile</code></summary>

```diff
@@ -44,3 +44,11 @@ stop:
 # Run the tests, and show which lines of the package api they ran. Extra arguments go to pytest: just cov tests/unit
 cov *args:
     uv run pytest --cov=api --cov-report=term-missing "$@"
+
+# Format the code, one way. Extra arguments go to ruff: just fmt --check, just fmt --diff
+fmt *args:
+    uv run ruff format "$@"
+
+# Check the code against the rules in pyproject.toml. Extra arguments go to ruff: just lint --fix
+lint *args:
+    uv run ruff check "$@"
```

</details>

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/b947593e84b71486902b3ff1c7f9e1458b70e3f7).

This part ends at the code at `b947593e84b7`, partway through `step-05`. [The code after this part](https://github.com/rahuldave/cheesecracker/tree/b947593e84b71486902b3ff1c7f9e1458b70e3f7)

<a id="step-05.5-to-step-05.7"></a>

## 5.5 to 5.7 The fixes

This walk goes through [rahuldave/cheesecracker](https://github.com/rahuldave/cheesecracker) on GitHub. This part starts from the code at `b947593e84b7`, partway through `step-05`. [The code before this part](https://github.com/rahuldave/cheesecracker/tree/b947593e84b71486902b3ff1c7f9e1458b70e3f7)

```bash
just setup
```

The formatter fixes the layout, and the linter removes the import that nothing uses. A person shortens the long line.

<a id="step-05.5"></a>

### 5.5 the formatter fixes the spaces, the blank lines and the line breaks

The formatter writes every file in one layout, so nobody fixes spaces by hand.

```bash
just fmt
```

ruff prints `2 files reformatted`. In `service.py`, `z=power(x,2)+power(y,2)` now has its spaces, and two blank lines
separate the routes. The long `return` now spreads its dictionary over six lines, one key on each line. In `utils.py`, the blank lines are two.

```bash
just fmt --check
```

ruff prints `6 files already formatted`.

Check your files against this move. The command prints nothing when they match, and otherwise shows what differs:

```bash
git check 42642115394d
```

If you are stuck, this command puts the files of this move in place of yours. A file that you made yourself stays:

```bash
git restore --source b947593e84b7 --staged -- :/ && git restore --source 42642115394d --staged --worktree -- :/
```

#### What changed

- <a id="step-05.5-src-api-service-api-service.py-1e1e71"></a>`src/api-service/api/service.py`: the spaces, the blank lines, and the long `return` on six lines. 10 lines added, 2 removed. It comes from `just fmt`. [The file](https://github.com/rahuldave/cheesecracker/blob/42642115394d8e19ead4c27bafaf4d90d991b0c9/src/api-service/api/service.py).
  - look at `z = power(x, 2) + power(y, 2)`
- <a id="step-05.5-src-api-service-api-utils.py-18f1f8"></a>`src/api-service/api/utils.py`: the space after the comma, and two blank lines before `unused_function()`. 1 line added, 2 removed. It comes from `just fmt`. [The file](https://github.com/rahuldave/cheesecracker/blob/42642115394d8e19ead4c27bafaf4d90d991b0c9/src/api-service/api/utils.py).
  - look at `def power(base, exponent):`

<a id="step-05.5-src-api-service-api-service.py-1e1e71-diff"></a>

<details><summary>The diff that <code>just fmt</code> makes in <code>src/api-service/api/service.py</code></summary>

```diff
@@ -20,10 +20,13 @@ app.add_middleware(
     allow_headers=["*"],
 )
 
+
 # Routes
 @app.get("/")
 async def get_index():
     return {"message": "Welcome to the Cheese App CI/CD Tutorial!"}
+
+
 @app.get("/health")
 async def health_check():
     return {"status": "healthy"}
@@ -34,6 +37,11 @@ async def calculate_euclidean_distance(x: float = 1, y: float = 2):
     """Calculate the Euclidean distance from origin: sqrt(x^2 + y^2)
     Returns the distance of a point (x, y) from the origin (0, 0)
     """
-    z=power(x,2)+power(y,2)
+    z = power(x, 2) + power(y, 2)
     result = math.sqrt(z)
-    return {"x": x, "y": y, "distance": result, "message": "This is a very long line that exceeds 120 characters and Black will complain about it so we need to fix this formatting issue"}
+    return {
+        "x": x,
+        "y": y,
+        "distance": result,
+        "message": "This is a very long line that exceeds 120 characters and Black will complain about it so we need to fix this formatting issue",
+    }
```

</details>

<a id="step-05.5-src-api-service-api-utils.py-18f1f8-diff"></a>

<details><summary>The diff that <code>just fmt</code> makes in <code>src/api-service/api/utils.py</code></summary>

```diff
@@ -4,11 +4,10 @@ Used by the Cheese App API Service
 """
 
 
-def power(base,exponent):
+def power(base, exponent):
     """Calculate base raised to the power of exponent"""
     return base**exponent
 
 
-
 def unused_function():
     pass
```

</details>

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/42642115394d8e19ead4c27bafaf4d90d991b0c9). [The code after it](https://github.com/rahuldave/cheesecracker/tree/42642115394d8e19ead4c27bafaf4d90d991b0c9).

<a id="step-05.6"></a>

### 5.6 the linter removes the import that nothing uses

ruff can remove an import that nothing uses, because the change cannot alter what the code does. The linter still fails
after the fix, because one error is left. `|| true` makes the command count as a success all the same.

```bash
just lint --fix || true
```

ruff removes the line `import os` from `service.py`, and still reports the long line.

```bash
just lint
```

ruff finds 1 error, `E501`, and fails. The line of the message is 147 characters long, and no tool can shorten words.

Check your files against this move. The command prints nothing when they match, and otherwise shows what differs:

```bash
git check cfd2ebb050dd
```

If you are stuck, this command puts the files of this move in place of yours. A file that you made yourself stays:

```bash
git restore --source 42642115394d --staged -- :/ && git restore --source cfd2ebb050dd --staged --worktree -- :/
```

#### What changed

- <a id="step-05.6-src-api-service-api-service.py-1e1e71"></a>`src/api-service/api/service.py`: the import of `os`, removed. 0 lines added, 1 removed. It comes from `just lint --fix || true`. [The file](https://github.com/rahuldave/cheesecracker/blob/cfd2ebb050dda3ce951f0f28e84b9ae09fbe14d5/src/api-service/api/service.py).

<a id="step-05.6-src-api-service-api-service.py-1e1e71-diff"></a>

<details><summary>The diff that <code>just lint --fix || true</code> makes in <code>src/api-service/api/service.py</code></summary>

```diff
@@ -5,7 +5,6 @@ Cheese App API Service
 from fastapi import FastAPI
 from starlette.middleware.cors import CORSMiddleware
 import math
-import os
 from api.utils import power
 
 # Setup FastAPI app
```

</details>

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/cfd2ebb050dda3ce951f0f28e84b9ae09fbe14d5). [The code after it](https://github.com/rahuldave/cheesecracker/tree/cfd2ebb050dda3ce951f0f28e84b9ae09fbe14d5).

<a id="step-05.7"></a>

### 5.7 a shorter line, by hand

A long message is a choice of words, so a person has to shorten it. The new message is short enough for one line.

Apply the patch to [`src/api-service/api/service.py`](#step-05.7-src-api-service-api-service.py-1e1e71), from the top of the project:

```bash
curl -fsSL https://github.com/rahuldave/cheesecracker/commit/693a713a458db01debdd7ebc3b084ed1b0350b08.diff | git apply --include=src/api-service/api/service.py
```

Apply the patch. It changes the value of `"message"` in the `return` at the end of the file.

```bash
just lint
```

ruff prints `All checks passed!`. Note that ruff never reports `unused_function()`. No rule of `E`, `W` or `F` looks for
a function that nothing calls.

Check your files against this move. The command prints nothing when they match, and otherwise shows what differs:

```bash
git check 693a713a458d
```

If you are stuck, this command puts the files of this move in place of yours. A file that you made yourself stays:

```bash
git restore --source cfd2ebb050dd --staged -- :/ && git restore --source 693a713a458d --staged --worktree -- :/
```

#### What changed

- <a id="step-05.7-src-api-service-api-service.py-1e1e71"></a>`src/api-service/api/service.py`: a shorter message. 1 line added, 1 removed. [The file](https://github.com/rahuldave/cheesecracker/blob/693a713a458db01debdd7ebc3b084ed1b0350b08/src/api-service/api/service.py).
  - look at `"message": "This is a very long line that exceeds 120 characters blah",`

<a id="step-05.7-src-api-service-api-service.py-1e1e71-diff"></a>

<details open><summary>The patch to <code>src/api-service/api/service.py</code></summary>

```diff
@@ -42,5 +42,5 @@ async def calculate_euclidean_distance(x: float = 1, y: float = 2):
         "x": x,
         "y": y,
         "distance": result,
-        "message": "This is a very long line that exceeds 120 characters and Black will complain about it so we need to fix this formatting issue",
+        "message": "This is a very long line that exceeds 120 characters blah",
     }
```

</details>

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/693a713a458db01debdd7ebc3b084ed1b0350b08).

This part ends at the code at `693a713a458d`, partway through `step-05`. [The code after this part](https://github.com/rahuldave/cheesecracker/tree/693a713a458db01debdd7ebc3b084ed1b0350b08)

<a id="step-05.8"></a>

## 5.8 One recipe for every check

This walk goes through [rahuldave/cheesecracker](https://github.com/rahuldave/cheesecracker) on GitHub. This part starts from the code at `693a713a458d`, partway through `step-05`. [The code before this part](https://github.com/rahuldave/cheesecracker/tree/693a713a458db01debdd7ebc3b084ed1b0350b08)

```bash
just setup
```

The layout, the rules and the tests each have a recipe. One recipe, `just check`, runs them all.

### 5.8 just check runs every check

The layout, the rules and the tests each have a recipe now. The recipe `check` runs all three, and stops at the
first that fails.

Apply the patch to [`justfile`](#step-05.8-justfile-424f0b), from the top of the project:

```bash
curl -fsSL https://github.com/rahuldave/cheesecracker/commit/e89e0cfa991257757c887f687087a9c77c093e20.diff | git apply --include=justfile
```

Read the recipe `check` at the end of the `justfile`.

```bash
just check
```

just prints each command, and the last line is `21 passed`.

Check your files against this move. The command prints nothing when they match, and otherwise shows what differs:

```bash
git check e89e0cfa9912
```

If you are stuck, this command puts the files of this move in place of yours. A file that you made yourself stays:

```bash
git fetch --quiet --tags --force origin; git restore --source 693a713a458d --staged -- :/ && git restore --source step-05 --staged --worktree -- :/
```

#### What changed

- <a id="step-05.8-justfile-424f0b"></a>`justfile`: the recipe `check`. 6 lines added. [The file](https://github.com/rahuldave/cheesecracker/blob/e89e0cfa991257757c887f687087a9c77c093e20/justfile).
  - look at `just fmt --check`

<a id="step-05.8-justfile-424f0b-diff"></a>

<details open><summary>The patch to <code>justfile</code></summary>

```diff
@@ -52,3 +52,9 @@ fmt *args:
 # Check the code against the rules in pyproject.toml. Extra arguments go to ruff: just lint --fix
 lint *args:
     uv run ruff check "$@"
+
+# Everything a commit must pass: the format, the lint and the tests
+check:
+    just fmt --check
+    just lint
+    just test -q
```

</details>

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/e89e0cfa991257757c887f687087a9c77c093e20).

This part ends at the code at `step-05`. [The code after this part](https://github.com/rahuldave/cheesecracker/tree/step-05)

<a id="step-06"></a>

## 6 Checks before every commit

This walk goes through [rahuldave/cheesecracker](https://github.com/rahuldave/cheesecracker) on GitHub. This part starts from the code at `step-05`. [The code before this part](https://github.com/rahuldave/cheesecracker/tree/step-05)

`just check` works only when someone runs it. pre-commit runs checks before every `git commit`.

```bash
just setup
```

`just check` works only when someone runs it. pre-commit runs checks before every `git commit`, and stops the commit
when a check fails. The checks are in `.pre-commit-config.yaml`.

<a id="step-06.1"></a>

### 6.1 pre-commit, in the group lint

[pre-commit](https://pre-commit.com/) runs checks on the files of a commit, before git makes it. pre-commit calls
each check a hook. git has hooks of its own, scripts that it runs before a commit, and a later move installs one of
them to start pre-commit.

`.pre-commit-config.yaml` names the repository that each hook comes from, and the exact version to use.
The first time pre-commit runs, it downloads each hook and installs it in a folder of its own. A failed check stops the commit. If a hook fixes a file, the commit stops
too, and the fixed file waits for `git add` before the next try.

pre-commit checks the code, and the server does not need it to run. So it goes into the group `lint`, with ruff.

```bash
uv add --group lint pre-commit
```

```bash
uv run pre-commit --version
```

uv prints the version of pre-commit.

Check your files against this move. The command prints nothing when they match, and otherwise shows what differs:

```bash
git check f3ad230bf911
```

If you are stuck, this command puts the files of this move in place of yours. A file that you made yourself stays:

```bash
git fetch --quiet --tags --force origin; git restore --source step-05 --staged -- :/ && git restore --source f3ad230bf911 --staged --worktree -- :/
```

#### What changed

- <a id="step-06.1-pyproject.toml-5d07e7"></a>`pyproject.toml`: pre-commit, in the group `lint`. 1 line added. It comes from `uv add --group lint pre-commit`. [The file](https://github.com/rahuldave/cheesecracker/blob/f3ad230bf911e0c446eb6752ec2718b910d9e851/pyproject.toml).
  - look at `"pre-commit>=4.7.0",`
- <a id="step-06.1-uv.lock-8356d8"></a>`uv.lock`: the exact versions of pre-commit, and of the packages that it needs. 149 lines added, 1 removed. It comes from `uv add --group lint pre-commit`. [The file](https://github.com/rahuldave/cheesecracker/blob/f3ad230bf911e0c446eb6752ec2718b910d9e851/uv.lock).

<a id="step-06.1-pyproject.toml-5d07e7-diff"></a>

<details><summary>The diff that <code>uv add --group lint pre-commit</code> makes in <code>pyproject.toml</code></summary>

```diff
@@ -17,6 +17,7 @@ default-groups = "all"
 
 [dependency-groups]
 lint = [
+    "pre-commit>=4.7.0",
     "ruff>=0.17.1",
 ]
 test = [
```

</details>

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/f3ad230bf911e0c446eb6752ec2718b910d9e851). [The code after it](https://github.com/rahuldave/cheesecracker/tree/f3ad230bf911e0c446eb6752ec2718b910d9e851).

<a id="step-06.2"></a>

### 6.2 the checks that run before every commit

pre-commit needs a list of checks. `.pre-commit-config.yaml` names two repositories of checks. The first,
pre-commit-hooks, checks any file. The second, ruff-pre-commit, runs ruff's linter and formatter with the settings of
`pyproject.toml`.

Apply the patch to [`.pre-commit-config.yaml`](#step-06.2-.pre-commit-config.yaml-e7d14d), from the top of the project:

```bash
curl -fsSL https://github.com/rahuldave/cheesecracker/commit/009d77653eadfde09e438656cde77b5eac489115.diff | git apply --include=.pre-commit-config.yaml
```

Apply the patch, and read the two entries under `repos`. Each has an address, a version after `rev`, and its checks
under `hooks`.

pre-commit checks the files that git tracks, as they are in your folder. It skips a file that git does not track yet.
The first run downloads the checks, which takes a minute.

```bash
uv run pre-commit run --all-files
```

pre-commit prints a line for each check, `Passed` or `Skipped`.

`.pre-commit-config.yaml` is new, and git does not track it yet. So `check yaml` says `(no files to check)Skipped`.
`check json` says `Skipped`, because the project has no JSON file.

`ruff check` checks only the Python files that git knows, which are those in a commit and those that are staged. A patch and an
edit by hand stage nothing. So if you made every change here with patches or by hand, git knows none of the Python
files, and `ruff check` says `(no files to check)Skipped`. A command that puts in the files of a part, at the top of a
part or after a gap, stages them. If you ran one, `ruff check` says `Passed`. `ruff format` says `Passed`.

Check your files against this move. The command prints nothing when they match, and otherwise shows what differs:

```bash
git check 009d77653ead
```

If you are stuck, this command puts the files of this move in place of yours. A file that you made yourself stays:

```bash
git restore --source f3ad230bf911 --staged -- :/ && git restore --source 009d77653ead --staged --worktree -- :/
```

#### What changed

- <a id="step-06.2-.pre-commit-config.yaml-e7d14d"></a>`.pre-commit-config.yaml`: the checks that run before every commit. New, 22 lines. [The file](https://github.com/rahuldave/cheesecracker/blob/009d77653eadfde09e438656cde77b5eac489115/.pre-commit-config.yaml).
  - look at `- id: ruff-check`

<a id="step-06.2-.pre-commit-config.yaml-e7d14d-diff"></a>

<details open><summary>The patch to <code>.pre-commit-config.yaml</code></summary>

```diff
@@ -0,0 +1,22 @@
+# The checks that run before every commit. `just hook` installs them as a git hook.
+# Run them on every file: uv run pre-commit run --all-files
+repos:
+  # Checks of any file
+  - repo: https://github.com/pre-commit/pre-commit-hooks
+    rev: v6.0.0
+    hooks:
+      - id: trailing-whitespace
+      - id: end-of-file-fixer
+      - id: check-yaml
+      - id: check-added-large-files
+        args: ["--maxkb=1000"]
+      - id: check-json
+      - id: check-merge-conflict
+      - id: detect-private-key
+
+  # ruff, with the settings in pyproject.toml: the linter, then the formatter
+  - repo: https://github.com/astral-sh/ruff-pre-commit
+    rev: v0.17.1
+    hooks:
+      - id: ruff-check
+      - id: ruff-format
```

</details>

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/009d77653eadfde09e438656cde77b5eac489115). [The code after it](https://github.com/rahuldave/cheesecracker/tree/009d77653eadfde09e438656cde77b5eac489115).

<a id="step-06.3"></a>

### 6.3 just hook installs the checks as a git hook

pre-commit runs before every commit only after it installs a hook in `.git/hooks/`. `just setup` never changes git,
so the hook has a recipe of its own, `hook`. You run it once in each clone.

Apply the patch to [`justfile`](#step-06.3-justfile-424f0b), from the top of the project:

```bash
curl -fsSL https://github.com/rahuldave/cheesecracker/commit/57ec2e24ecd4946823edbfad27bc2296f80e3847.diff | git apply --include=justfile
```

Read the recipe `hook`, after the recipe `setup`.

```bash
just hook
```

pre-commit prints `pre-commit installed at .git/hooks/pre-commit`. From now on, `git commit` runs the checks first.

Check your files against this move. The command prints nothing when they match, and otherwise shows what differs:

```bash
git check 57ec2e24ecd4
```

If you are stuck, this command puts the files of this move in place of yours. A file that you made yourself stays:

```bash
git fetch --quiet --tags --force origin; git restore --source 009d77653ead --staged -- :/ && git restore --source step-06 --staged --worktree -- :/
```

#### What changed

- <a id="step-06.3-justfile-424f0b"></a>`justfile`: the recipe `hook`. 4 lines added. [The file](https://github.com/rahuldave/cheesecracker/blob/57ec2e24ecd4946823edbfad27bc2296f80e3847/justfile).
  - look at `uv run pre-commit install`

<a id="step-06.3-justfile-424f0b-diff"></a>

<details open><summary>The patch to <code>justfile</code></summary>

```diff
@@ -11,6 +11,10 @@ default:
 setup:
     uv sync --locked
 
+# Install the git hook: the checks of .pre-commit-config.yaml before every commit. Once per clone
+hook:
+    uv run pre-commit install
+
 # Run the tests. Extra arguments go to pytest: just test tests/unit
 test *args:
     uv run pytest "$@"
```

</details>

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/57ec2e24ecd4946823edbfad27bc2296f80e3847).

This part ends at the code at `step-06`. [The code after this part](https://github.com/rahuldave/cheesecracker/tree/step-06)
