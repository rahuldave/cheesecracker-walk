<!-- Made by the walk skill's render.py from the facts of the walk. Do not edit it by hand: run just walk-update again. -->

# uv and the groups of dependencies of cheesecracker

**Build it yourself.** Every command to run and every change to make by hand, to make the project yourself. [Read the tutorial](read.md) is the other way to read it.

Each part ends with a section called **What changed**. It lists every file that the part changes, with a link to the file, and then the whole diff of each file, folded. A lockfile and a binary file have a link alone. The text above it explains the change. What changed is what you check your files against.

In Build it yourself, a file that a command makes says which command. A diff that is open is a patch. Apply it with the command in the text, or make the same change in your editor.

We follow the dependencies of cheesecracker, the API server of the Cheese App. uv installs Python and the packages of
the project, and writes their exact versions into one lockfile, `uv.lock`. The packages come in groups. The server
needs some of them to run. The tests and the checks need the others. This tutorial takes these parts out of the whole
tutorial of cheesecracker, and ends with the two Docker images, which install different groups.

We make the project in moves. A move is one small change, made by one patch or by one command. You make each move,
and the page checks your files against it.

Some changes of the whole tutorial are not in this one. After such a gap, the next section says what those changes
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

## 0 The project, and its `.venv`

The project starts as a `pyproject.toml`, a `.python-version` and a lockfile. `just setup` makes the environment,
`.venv`, from the lockfile.

```bash
just setup
```

The project is empty. It has a `pyproject.toml` with no dependencies, an empty `uv.lock`, a `README.md`, a
`.gitignore`, and a `justfile` with one recipe, `setup`. `just setup` runs `uv sync --locked`. uv is the tool that installs Python and
packages. The command makes the folder `.venv`, with Python 3.12 and every package of `uv.lock` in it. Run it again.

```bash
just setup
```

uv prints `Resolved 1 package` and `Checked`, and no line `Creating virtual environment`. A second run changes
nothing.

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
git rm -rq --cached --ignore-unmatch -- :/ && git restore --source cf50a6ef5045 --staged --worktree -- :/
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

<details><summary>The diff of <code>.python-version</code></summary>

```diff
@@ -0,0 +1 @@
+3.12
```

</details>

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

[The code](https://github.com/rahuldave/cheesecracker/tree/cf50a6ef5045e08ca727aeaffdb710fad2cae096).

<a id="step-01.2"></a>

## 1.2 A group for the tests

This part does not start where the part before it ends, because the walk skips some changes. This command puts the files that this part starts from in place of yours. Run it, and then go on:

```bash
git restore --source cf50a6ef5045 --staged -- :/ && git restore --source bb1701fd54ba --staged --worktree -- :/
```

```bash
just setup
```

The whole tutorial first writes the function `power()`, in `src/api-service/api/utils.py`. Its tests need pytest, and
the server never does. So pytest goes into a group of its own.

[The code before this part](https://github.com/rahuldave/cheesecracker/tree/bb1701fd54ba7c6aa9bd971c6651f34439f004d6). [The code after it](https://github.com/rahuldave/cheesecracker/tree/fc8c3bce83604277393460c4eb243d176d51905c).

### 1.2 pytest, in the dependency group test

A test calls `power()` and compares the result with the answer that we expect. pytest finds the tests of a project,
runs them, and reports each failure. The tests need pytest, but the server does not. So pytest goes into a group of
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

<a id="step-02.1"></a>

## 2.1 The dependencies of the server

This part does not start where the part before it ends, because the walk skips some changes. This command puts the files that this part starts from in place of yours. Run it, and then go on:

```bash
git restore --source fc8c3bce8360 --staged -- :/ && git restore --source e86e0c3d10bb --staged --worktree -- :/
```

```bash
just setup
```

The whole tutorial then tells pytest where the tests are, adds eleven unit tests of `power()`, and the recipe
`just test`. Next comes the server, which needs FastAPI to build it and uvicorn to run it. They are dependencies of the project itself, and of no
group.

[The code before this part](https://github.com/rahuldave/cheesecracker/tree/e86e0c3d10bbe9409674f1c65ce4afdc38e7f76f). [The code after it](https://github.com/rahuldave/cheesecracker/tree/18dd9dfb1c9597e50a8b404a6cb6379c1c149320).

### 2.1 FastAPI, and uvicorn to serve it

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
git restore --source e86e0c3d10bb --staged -- :/ && git restore --source 18dd9dfb1c95 --staged --worktree -- :/
```

#### What changed

- <a id="step-02.1-pyproject.toml-5d07e7"></a>`pyproject.toml`: FastAPI and uvicorn, the dependencies of the server. 4 lines added, 1 removed. It comes from `uv add fastapi uvicorn`. [The file](https://github.com/rahuldave/cheesecracker/blob/18dd9dfb1c9597e50a8b404a6cb6379c1c149320/pyproject.toml).
  - look at `"fastapi>=0.143.2",`
- <a id="step-02.1-uv.lock-8356d8"></a>`uv.lock`: the exact versions of FastAPI, uvicorn, and the packages that they need. 263 lines added. It comes from `uv add fastapi uvicorn`. [The file](https://github.com/rahuldave/cheesecracker/blob/18dd9dfb1c9597e50a8b404a6cb6379c1c149320/uv.lock).

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

<a id="step-03.2"></a>

## 3.2 More tools for the tests

This part does not start where the part before it ends, because the walk skips some changes. This command puts the files that this part starts from in place of yours. Run it, and then go on:

```bash
git restore --source 18dd9dfb1c95 --staged -- :/ && git restore --source e576e7bb9e55 --staged --worktree -- :/
```

```bash
just setup
```

The whole tutorial then writes the app, the recipes `just serve` and `just stop`, and an endpoint that computes a
distance. The integration tests need two more packages, and they go into the group `test`.

[The code before this part](https://github.com/rahuldave/cheesecracker/tree/e576e7bb9e550b1a572a764b15f8d0bfde229d97). [The code after it](https://github.com/rahuldave/cheesecracker/tree/3c75ffb1a089431a2455a69e9bcb3ec0302224f0).

### 3.2 httpx2 and pytest-asyncio, for tests of the app

FastAPI's `TestClient` sends requests to the app in the same process, with no server. It needs the package httpx2.
One test of the app is a coroutine, an `async` function, and pytest-asyncio runs it.

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

<a id="step-04.1"></a>

## 4.1 One more tool for the tests

This part does not start where the part before it ends, because the walk skips some changes. This command puts the files that this part starts from in place of yours. Run it, and then go on:

```bash
git restore --source 3c75ffb1a089 --staged -- :/ && git restore --source 5862780b44d2 --staged --worktree -- :/
```

```bash
just setup
```

The whole tutorial then adds ten integration tests. The package pytest-cov measures which lines of the code the tests
run, and it goes into the group `test`.

[The code before this part](https://github.com/rahuldave/cheesecracker/tree/5862780b44d2f4ec87088c971dec57f23c11139e). [The code after it](https://github.com/rahuldave/cheesecracker/tree/54de20e8055ec62726d8956cac3a944433dc404c).

### 4.1 pytest-cov, to measure what the tests run

pytest-cov adds the option `--cov` to pytest. It is a tool of the tests, so it goes into the group `test`.

```bash
uv add --group test pytest-cov
```

The check runs the unit tests alone, with coverage of the package `api`.

```bash
uv run pytest tests/unit --cov=api -q
```

The table shows `utils.py` at 100% and `service.py` at 0%. The unit tests call `power()`, and never the app.

Check your files against this move. The command prints nothing when they match, and otherwise shows what differs:

```bash
git check 54de20e8055e
```

If you are stuck, this command puts the files of this move in place of yours. A file that you made yourself stays:

```bash
git restore --source 5862780b44d2 --staged -- :/ && git restore --source 54de20e8055e --staged --worktree -- :/
```

#### What changed

- <a id="step-04.1-pyproject.toml-5d07e7"></a>`pyproject.toml`: pytest-cov, in the group `test`. 1 line added. It comes from `uv add --group test pytest-cov`. [The file](https://github.com/rahuldave/cheesecracker/blob/54de20e8055ec62726d8956cac3a944433dc404c/pyproject.toml).
  - look at `"pytest-cov>=7.1.0",`
- <a id="step-04.1-uv.lock-8356d8"></a>`uv.lock`: the exact versions of pytest-cov, and of coverage, which it uses. 115 lines added. It comes from `uv add --group test pytest-cov`. [The file](https://github.com/rahuldave/cheesecracker/blob/54de20e8055ec62726d8956cac3a944433dc404c/uv.lock).

<details><summary>The diff that <code>uv add --group test pytest-cov</code> makes in <code>pyproject.toml</code></summary>

```diff
@@ -20,6 +20,7 @@ test = [
     "httpx2>=2.13.1",
     "pytest>=9.1.1",
     "pytest-asyncio>=1.4.0",
+    "pytest-cov>=7.1.0",
 ]
 
 [tool.pytest.ini_options]
```

</details>

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/54de20e8055ec62726d8956cac3a944433dc404c). [The code after it](https://github.com/rahuldave/cheesecracker/tree/54de20e8055ec62726d8956cac3a944433dc404c).

<a id="step-05.2"></a>

## 5.2 A second group, for the checks

This part does not start where the part before it ends, because the walk skips some changes. This command puts the files that this part starts from in place of yours. Run it, and then go on:

```bash
git restore --source 54de20e8055e --staged -- :/ && git restore --source f77650600c11 --staged --worktree -- :/
```

```bash
just setup
```

The whole tutorial then adds the recipe `just cov`, and a change made in a hurry. ruff checks the code, and the server
does not need it. So ruff starts a second group, `lint`.

[The code before this part](https://github.com/rahuldave/cheesecracker/tree/f77650600c1176caa9aaee86e6cddfccc43fc69c). [The code after it](https://github.com/rahuldave/cheesecracker/tree/9662828462ba2a21ca04a79c412e83ac070767f6).

### 5.2 ruff, the formatter and the linter, in the group lint

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

<a id="step-06.1"></a>

## 6.1 pre-commit, in the group lint

This part does not start where the part before it ends, because the walk skips some changes. This command puts the files that this part starts from in place of yours. Run it, and then go on:

```bash
git restore --source 9662828462ba --staged -- :/ && git restore --source e89e0cfa9912 --staged --worktree -- :/
```

```bash
just setup
```

The whole tutorial then sets up ruff, and fixes the code with it. pre-commit runs the checks before every commit, so
it joins ruff in the group `lint`.

[The code before this part](https://github.com/rahuldave/cheesecracker/tree/e89e0cfa991257757c887f687087a9c77c093e20). [The code after it](https://github.com/rahuldave/cheesecracker/tree/f3ad230bf911e0c446eb6752ec2718b910d9e851).

### 6.1 pre-commit, in the group lint

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
git restore --source e89e0cfa9912 --staged -- :/ && git restore --source f3ad230bf911 --staged --worktree -- :/
```

#### What changed

- <a id="step-06.1-pyproject.toml-5d07e7"></a>`pyproject.toml`: pre-commit, in the group `lint`. 1 line added. It comes from `uv add --group lint pre-commit`. [The file](https://github.com/rahuldave/cheesecracker/blob/f3ad230bf911e0c446eb6752ec2718b910d9e851/pyproject.toml).
  - look at `"pre-commit>=4.7.0",`
- <a id="step-06.1-uv.lock-8356d8"></a>`uv.lock`: the exact versions of pre-commit, and of the packages that it needs. 149 lines added, 1 removed. It comes from `uv add --group lint pre-commit`. [The file](https://github.com/rahuldave/cheesecracker/blob/f3ad230bf911e0c446eb6752ec2718b910d9e851/uv.lock).

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

<a id="step-07.2"></a>

## 7.2 The run image installs no group

This part does not start where the part before it ends, because the walk skips some changes. This command puts the files that this part starts from in place of yours. Run it, and then go on:

```bash
git restore --source f3ad230bf911 --staged -- :/ && git restore --source 2a28a5c704d6 --staged --worktree -- :/
```

```bash
just setup
```

The whole tutorial then writes the checks of pre-commit, and `.dockerignore`. The run image is the server as it runs
in production. It needs the dependencies of the project, and no group.

[The code before this part](https://github.com/rahuldave/cheesecracker/tree/2a28a5c704d6d9ed8200aeb8962aa01e5c2ac552). [The code after it](https://github.com/rahuldave/cheesecracker/tree/0acb4556f724cd94e4a71ab4fff739a974413fc0).

### 7.2 the run image, the server as it runs in production

The run image needs Python, the dependencies of the server, and the code. It does not need uv, the tests, ruff or
pre-commit. The `Dockerfile` builds the image in stages, and the run image takes only what the server needs.

Apply the patch to [`Dockerfile`](#step-07.2-Dockerfile-6651dd), from the top of the project:

```bash
curl -fsSL https://github.com/rahuldave/cheesecracker/commit/0acb4556f724cd94e4a71ab4fff739a974413fc0.diff | git apply --include=Dockerfile
```

Apply the patch, and read the three stages, which start at each `FROM`. Read the line `RUN uv sync` in the stage
`build`, and the line `COPY --from=build` and the `CMD` in the stage `run`.

```bash
docker build --target run --tag cheesecracker-run .
```

Docker prints each line of the `Dockerfile` as it runs. Near the end, the line `naming to
docker.io/library/cheesecracker-run` names the image.

```bash
docker image ls cheesecracker-run
```

The image is about 173 MB, nearly all of it Python and the dependencies of the server.

Check your files against this move. The command prints nothing when they match, and otherwise shows what differs:

```bash
git check 0acb4556f724
```

If you are stuck, this command puts the files of this move in place of yours. A file that you made yourself stays:

```bash
git restore --source 2a28a5c704d6 --staged -- :/ && git restore --source 0acb4556f724 --staged --worktree -- :/
```

#### What changed

- <a id="step-07.2-Dockerfile-6651dd"></a>`Dockerfile`: the stages `uv`, `build` and `run`. New, 34 lines. [The file](https://github.com/rahuldave/cheesecracker/blob/0acb4556f724cd94e4a71ab4fff739a974413fc0/Dockerfile).
  - look at `RUN uv sync --locked --no-default-groups`

<details open><summary>The patch to <code>Dockerfile</code></summary>

```diff
@@ -0,0 +1,34 @@
+# The images of the API server, built from uv.lock, so they hold the versions that the lockfile names.
+#
+#   docker build --target run --tag cheesecracker-run .
+#       the production image: Python, the dependencies of the server and its code. It starts the server
+#
+# The environment lives in /opt/venv, never in the project folder.
+
+# uv itself, copied from its own image
+FROM ghcr.io/astral-sh/uv:0.12.13 AS uv
+
+# The environment of the server, without the dependency groups: no tests, no ruff, no pre-commit
+FROM python:3.12-slim-bookworm AS build
+COPY --from=uv /uv /usr/local/bin/uv
+ENV UV_PROJECT_ENVIRONMENT=/opt/venv \
+    UV_PYTHON_DOWNLOADS=never \
+    UV_COMPILE_BYTECODE=1 \
+    UV_LINK_MODE=copy \
+    UV_NO_CACHE=1
+WORKDIR /app
+COPY pyproject.toml uv.lock .python-version ./
+RUN uv sync --locked --no-default-groups
+
+# The run image: Python, the environment from the build stage, and the code. No uv, and no dev tools
+FROM python:3.12-slim-bookworm AS run
+RUN useradd --create-home --uid 1000 app
+COPY --from=build /opt/venv /opt/venv
+WORKDIR /app
+COPY src/api-service/api ./api
+ENV PATH=/opt/venv/bin:$PATH \
+    PYTHONUNBUFFERED=1
+USER app
+EXPOSE 9000
+# Four workers, and no restart on a change: the server as it runs in production
+CMD ["uvicorn", "api.service:app", "--host", "0.0.0.0", "--port", "9000", "--workers", "4"]
```

</details>

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/0acb4556f724cd94e4a71ab4fff739a974413fc0). [The code after it](https://github.com/rahuldave/cheesecracker/tree/0acb4556f724cd94e4a71ab4fff739a974413fc0).

<a id="step-08.1"></a>

## 8.1 One more tool for the tests, after the run image

This part does not start where the part before it ends, because the walk skips some changes. This command puts the files that this part starts from in place of yours. Run it, and then go on:

```bash
git restore --source 0acb4556f724 --staged -- :/ && git restore --source d444e0387d25 --staged --worktree -- :/
```

```bash
just setup
```

The whole tutorial then adds recipes for Docker. The system tests send real HTTP requests, with the package `requests`.
`requests` goes into the group `test`, and the run image never sees it.

[The code before this part](https://github.com/rahuldave/cheesecracker/tree/d444e0387d256c62024581e98f59d670cbc93296). [The code after it](https://github.com/rahuldave/cheesecracker/tree/de0780b72a5cad4f0ed70e9c0cdcdee9d1028562).

### 8.1 requests, for tests that send real HTTP requests

The system tests send their requests with the package requests. It is a tool of the tests, so it goes into the group
`test`.

```bash
uv add --group test requests
```

```bash
uv run python -c "import requests; print(requests.__version__)"
```

Python prints the version of requests.

Check your files against this move. The command prints nothing when they match, and otherwise shows what differs:

```bash
git check de0780b72a5c
```

If you are stuck, this command puts the files of this move in place of yours. A file that you made yourself stays:

```bash
git restore --source d444e0387d25 --staged -- :/ && git restore --source de0780b72a5c --staged --worktree -- :/
```

#### What changed

- <a id="step-08.1-pyproject.toml-5d07e7"></a>`pyproject.toml`: requests, in the group `test`. 1 line added. It comes from `uv add --group test requests`. [The file](https://github.com/rahuldave/cheesecracker/blob/de0780b72a5cad4f0ed70e9c0cdcdee9d1028562/pyproject.toml).
  - look at `"requests>=2.34.2",`
- <a id="step-08.1-uv.lock-8356d8"></a>`uv.lock`: the exact versions of requests, and of the packages that it needs. 166 lines added. It comes from `uv add --group test requests`. [The file](https://github.com/rahuldave/cheesecracker/blob/de0780b72a5cad4f0ed70e9c0cdcdee9d1028562/uv.lock).

<details><summary>The diff that <code>uv add --group test requests</code> makes in <code>pyproject.toml</code></summary>

```diff
@@ -25,6 +25,7 @@ test = [
     "pytest>=9.1.1",
     "pytest-asyncio>=1.4.0",
     "pytest-cov>=7.1.0",
+    "requests>=2.34.2",
 ]
 
 [tool.pytest.ini_options]
```

</details>

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/de0780b72a5cad4f0ed70e9c0cdcdee9d1028562). [The code after it](https://github.com/rahuldave/cheesecracker/tree/de0780b72a5cad4f0ed70e9c0cdcdee9d1028562).

<a id="step-09.1"></a>

## 9.1 The dev image installs every group

This part does not start where the part before it ends, because the walk skips some changes. This command puts the files that this part starts from in place of yours. Run it, and then go on:

```bash
git restore --source de0780b72a5c --staged -- :/ && git restore --source 277d54eebce4 --staged --worktree -- :/
```

```bash
just setup
```

The whole tutorial then adds the system tests, and the recipe `just test-system`. The dev image is for working on the
project, so it installs every group.

[The code before this part](https://github.com/rahuldave/cheesecracker/tree/277d54eebce4a1727ed3d8a5c46570ca4a95f75d). [The code after it](https://github.com/rahuldave/cheesecracker/tree/ccf62d330f28c6eec4c0b993e1a71e2c11a8b5af).

### 9.1 the dev image, to work on the project in Linux

The dev image is a fourth stage of the `Dockerfile`, `dev`. It takes uv from the stage `uv`, and nothing from the
run image.

Apply the patch to [`Dockerfile`](#step-09.1-Dockerfile-6651dd), from the top of the project:

```bash
curl -fsSL https://github.com/rahuldave/cheesecracker/commit/ccf62d330f28c6eec4c0b993e1a71e2c11a8b5af.diff | git apply --include=Dockerfile
```

Apply the patch. Read the new lines at the top of the file, then the stage `dev` at the end. Read its line `RUN uv
tool install`, and its line `RUN uv sync --locked`, which installs every group.

```bash
docker build --target dev --tag cheesecracker-dev .
```

Near the end, the line `naming to docker.io/library/cheesecracker-dev` names the image.

```bash
docker image ls cheesecracker-dev
```

The image is about 353 MB, twice the run image, because it has every tool.

Check your files against this move. The command prints nothing when they match, and otherwise shows what differs:

```bash
git check ccf62d330f28
```

If you are stuck, this command puts the files of this move in place of yours. A file that you made yourself stays:

```bash
git restore --source 277d54eebce4 --staged -- :/ && git restore --source ccf62d330f28 --staged --worktree -- :/
```

#### What changed

- <a id="step-09.1-Dockerfile-6651dd"></a>`Dockerfile`: the stage `dev`, and its lines in the comment at the top. 26 lines added. [The file](https://github.com/rahuldave/cheesecracker/blob/ccf62d330f28c6eec4c0b993e1a71e2c11a8b5af/Dockerfile).
  - look at `FROM python:3.12-slim-bookworm AS dev`

<details open><summary>The patch to <code>Dockerfile</code></summary>

```diff
@@ -2,6 +2,8 @@
 #
 #   docker build --target run --tag cheesecracker-run .
 #       the production image: Python, the dependencies of the server and its code. It starts the server
+#   docker build --target dev --tag cheesecracker-dev .
+#       the development image: every dependency group, uv, just and git. It starts a shell
 #
 # The environment lives in /opt/venv, never in the project folder.
 
@@ -32,3 +34,27 @@ USER app
 EXPOSE 9000
 # Four workers, and no restart on a change: the server as it runs in production
 CMD ["uvicorn", "api.service:app", "--host", "0.0.0.0", "--port", "9000", "--workers", "4"]
+
+# The dev image: the whole project with every dependency group, and uv, just and git to work in it
+FROM python:3.12-slim-bookworm AS dev
+# hadolint ignore=DL3008
+RUN apt-get update \
+    && apt-get install --yes --no-install-recommends ca-certificates curl git \
+    && rm -rf /var/lib/apt/lists/*
+COPY --from=uv /uv /uvx /usr/local/bin/
+ENV UV_PROJECT_ENVIRONMENT=/opt/venv \
+    UV_PYTHON_DOWNLOADS=never \
+    UV_LINK_MODE=copy \
+    UV_NO_CACHE=1 \
+    UV_TOOL_BIN_DIR=/usr/local/bin \
+    PATH=/opt/venv/bin:$PATH \
+    PYTHONUNBUFFERED=1
+# just, as a wheel from PyPI. git refuses a folder that another user owns, and a mounted folder
+# belongs to the user of the host, so every folder is safe
+RUN uv tool install "rust-just==1.50.0" \
+    && git config --system --add safe.directory '*'
+WORKDIR /work
+COPY pyproject.toml uv.lock .python-version ./
+RUN uv sync --locked
+COPY . .
+CMD ["bash"]
```

</details>

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/ccf62d330f28c6eec4c0b993e1a71e2c11a8b5af). [The code after it](https://github.com/rahuldave/cheesecracker/tree/ccf62d330f28c6eec4c0b993e1a71e2c11a8b5af).
