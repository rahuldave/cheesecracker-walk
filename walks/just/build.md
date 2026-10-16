<!-- Made by the walk skill's render.py from the facts of the walk. Do not edit it by hand: run just walk-update again. -->

# just, the recipes of cheesecracker

**Build it yourself.** Every command to run and every change to make by hand, to make the project yourself. [Read the tutorial](read.md) is the other way to read it.

Each part ends with a section called **What changed**. It lists every file that the part changes, with a link to the file, and then the whole diff of each file, folded. A lockfile and a binary file have a link alone. The text above it explains the change. What changed is what you check your files against.

In Build it yourself, a file that a command makes says which command. A diff that is open is a patch. Apply it with the command in the text, or make the same change in your editor.

We follow the `justfile` of cheesecracker, the API server of the Cheese App. The program `just` reads this file. The
file gives each task of the project a short name, and each named task is a recipe.

The recipes keep the commands in one place, so a person, a git hook and Docker run the same
thing. `just` alone lists them, and `just --show NAME` prints one. This tutorial takes the recipes out of the whole
tutorial of cheesecracker, one at a time, as each one comes in. Every recipe here is in the `justfile` of the project
cheesecracker, in the folder of its code.

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

Then add the command `git check`, which compares your files with the project's after each move, and leaves what `git add` has staged as it was.

```bash
git config alias.check '!f() { i="$(git rev-parse --git-path check-index)" && { cp -p "$(git rev-parse --git-path index)" "$i" 2>/dev/null || rm -f "$i"; } && export GIT_INDEX_FILE="$i" && git add -A && git diff --cached --stat "$1"; }; f'
```

<a id="step-00"></a>

## 0 The project, and its justfile

This walk goes through [rahuldave/cheesecracker](https://github.com/rahuldave/cheesecracker) on GitHub. This part is where the project starts, at `step-00`. [The code of this part](https://github.com/rahuldave/cheesecracker/tree/step-00)

The first recipe, `setup`, makes the environment of the project.

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

This part ends at the code at `step-00`. [The code after this part](https://github.com/rahuldave/cheesecracker/tree/step-00)

<a id="step-01.5"></a>

## 1.5 just test

This walk goes through [rahuldave/cheesecracker](https://github.com/rahuldave/cheesecracker) on GitHub. This part starts from the code at `cbc98b162671`, partway through `step-01`. [The code before this part](https://github.com/rahuldave/cheesecracker/tree/cbc98b162671bce7a7ba82d7d18c7dc6d530b746)

This part does not start where the part before it ends, because the walk skips some changes. This command puts the files that this part starts from in place of yours. Run it, and then go on:

```bash
git restore --source cf50a6ef5045 --staged -- :/ && git restore --source cbc98b162671 --staged --worktree -- :/
```

```bash
just setup
```

The whole tutorial first writes the function `power()`, adds pytest, tells pytest where the tests are, and writes
eleven unit tests. The command that runs them gets a recipe.

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
git restore --source cbc98b162671 --staged -- :/ && git restore --source e86e0c3d10bb --staged --worktree -- :/
```

#### What changed

- <a id="step-01.5-justfile-424f0b"></a>`justfile`: the recipe `test`. 4 lines added. [The file](https://github.com/rahuldave/cheesecracker/blob/e86e0c3d10bbe9409674f1c65ce4afdc38e7f76f/justfile).
  - look at `uv run pytest "$@"`

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

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/e86e0c3d10bbe9409674f1c65ce4afdc38e7f76f). [The code after it](https://github.com/rahuldave/cheesecracker/tree/step-01).

This part ends at the code at `step-01`. [The code after this part](https://github.com/rahuldave/cheesecracker/tree/step-01)

<a id="step-02.3"></a>

## 2.3 just serve and just stop

This walk goes through [rahuldave/cheesecracker](https://github.com/rahuldave/cheesecracker) on GitHub. This part starts from the code at `acee14f7dff4`, partway through `step-02`. [The code before this part](https://github.com/rahuldave/cheesecracker/tree/acee14f7dff43d6c5b76234b3a016985d74c60d2)

This part does not start where the part before it ends, because the walk skips some changes. This command puts the files that this part starts from in place of yours. Run it, and then go on:

```bash
git restore --source e86e0c3d10bb --staged -- :/ && git restore --source acee14f7dff4 --staged --worktree -- :/
```

```bash
just setup
```

The whole tutorial then adds FastAPI and uvicorn, and writes the app. Two recipes start the server and stop it.

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

The server answers `{"status":"healthy"}`. Open `http://localhost:9000/docs` in a browser to see the pages that
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
git restore --source acee14f7dff4 --staged -- :/ && git restore --source 69c3bf65c78c --staged --worktree -- :/
```

#### What changed

- <a id="step-02.3-.gitignore-a5cc29"></a>`.gitignore`: the log of the server and the number of its process. 4 lines added. [The file](https://github.com/rahuldave/cheesecracker/blob/69c3bf65c78cf8e96bf8f9ea1dde29279db1351a/.gitignore).
  - look at `.serve.pid`
- <a id="step-02.3-justfile-424f0b"></a>`justfile`: the recipes `serve` and `stop`. 26 lines added. [The file](https://github.com/rahuldave/cheesecracker/blob/69c3bf65c78cf8e96bf8f9ea1dde29279db1351a/justfile).
  - look at `uv run uvicorn api.service:app --app-dir src/api-service --host 0.0.0.0 --port 9000 --reload > .serve.log 2>&1 &`

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

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/69c3bf65c78cf8e96bf8f9ea1dde29279db1351a). [The code after it](https://github.com/rahuldave/cheesecracker/tree/step-02).

This part ends at the code at `step-02`. [The code after this part](https://github.com/rahuldave/cheesecracker/tree/step-02)

<a id="step-04.2"></a>

## 4.2 just cov

This walk goes through [rahuldave/cheesecracker](https://github.com/rahuldave/cheesecracker) on GitHub. This part starts from the code at `54de20e8055e`, partway through `step-04`. [The code before this part](https://github.com/rahuldave/cheesecracker/tree/54de20e8055ec62726d8956cac3a944433dc404c)

This part does not start where the part before it ends, because the walk skips some changes. This command puts the files that this part starts from in place of yours. Run it, and then go on:

```bash
git restore --source 69c3bf65c78c --staged -- :/ && git restore --source 54de20e8055e --staged --worktree -- :/
```

```bash
just setup
```

The whole tutorial then adds an endpoint, ten integration tests, and pytest-cov. Coverage gets a recipe.

### 4.2 just cov shows which lines the tests run

The recipe `cov` runs every test with coverage, and adds the column `Missing`, with the numbers of the lines that no
test ran.

Apply the patch to [`justfile`](#step-04.2-justfile-424f0b), from the top of the project:

```bash
curl -fsSL https://github.com/rahuldave/cheesecracker/commit/bfb9c783c67c31254ce8136784738c0f2fc17fb5.diff | git apply --include=justfile
```

Read the recipe `cov` at the end of the `justfile`.

```bash
just cov -q
```

The total is 95%. `service.py` misses line 31, the answer of `/health`, because no test asks for it.

Check your files against this move. The command prints nothing when they match, and otherwise shows what differs:

```bash
git check bfb9c783c67c
```

If you are stuck, this command puts the files of this move in place of yours. A file that you made yourself stays:

```bash
git restore --source 54de20e8055e --staged -- :/ && git restore --source bfb9c783c67c --staged --worktree -- :/
```

#### What changed

- <a id="step-04.2-justfile-424f0b"></a>`justfile`: the recipe `cov`. 4 lines added. [The file](https://github.com/rahuldave/cheesecracker/blob/bfb9c783c67c31254ce8136784738c0f2fc17fb5/justfile).
  - look at `uv run pytest --cov=api --cov-report=term-missing "$@"`

<details open><summary>The patch to <code>justfile</code></summary>

```diff
@@ -40,3 +40,7 @@ stop:
     done
     echo "stop: the server still answers after 30 seconds" >&2
     exit 1
+
+# Run the tests, and show which lines of the package api they ran. Extra arguments go to pytest: just cov tests/unit
+cov *args:
+    uv run pytest --cov=api --cov-report=term-missing "$@"
```

</details>

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/bfb9c783c67c31254ce8136784738c0f2fc17fb5). [The code after it](https://github.com/rahuldave/cheesecracker/tree/step-04).

This part ends at the code at `step-04`. [The code after this part](https://github.com/rahuldave/cheesecracker/tree/step-04)

<a id="step-05.4"></a>

## 5.4 just fmt and just lint

This walk goes through [rahuldave/cheesecracker](https://github.com/rahuldave/cheesecracker) on GitHub. This part starts from the code at `7c1670fd52a0`, partway through `step-05`. [The code before this part](https://github.com/rahuldave/cheesecracker/tree/7c1670fd52a0f62f414f31d415d10f2efb5a8405)

This part does not start where the part before it ends, because the walk skips some changes. This command puts the files that this part starts from in place of yours. Run it, and then go on:

```bash
git restore --source bfb9c783c67c --staged -- :/ && git restore --source 7c1670fd52a0 --staged --worktree -- :/
```

```bash
just setup
```

The whole tutorial then makes a change in a hurry, adds ruff, and gives it the rules of flake8. The formatter and the
linter get a recipe each.

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

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/b947593e84b71486902b3ff1c7f9e1458b70e3f7). [The code after it](https://github.com/rahuldave/cheesecracker/tree/b947593e84b71486902b3ff1c7f9e1458b70e3f7).

This part ends at the code at `b947593e84b7`, partway through `step-05`. [The code after this part](https://github.com/rahuldave/cheesecracker/tree/b947593e84b71486902b3ff1c7f9e1458b70e3f7)

<a id="step-05.8"></a>

## 5.8 just check

This walk goes through [rahuldave/cheesecracker](https://github.com/rahuldave/cheesecracker) on GitHub. This part starts from the code at `693a713a458d`, partway through `step-05`. [The code before this part](https://github.com/rahuldave/cheesecracker/tree/693a713a458db01debdd7ebc3b084ed1b0350b08)

This part does not start where the part before it ends, because the walk skips some changes. This command puts the files that this part starts from in place of yours. Run it, and then go on:

```bash
git restore --source b947593e84b7 --staged -- :/ && git restore --source 693a713a458d --staged --worktree -- :/
```

```bash
just setup
```

The whole tutorial then fixes the code with ruff, and shortens a long line. One recipe runs every check.

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
git restore --source 693a713a458d --staged -- :/ && git restore --source e89e0cfa9912 --staged --worktree -- :/
```

#### What changed

- <a id="step-05.8-justfile-424f0b"></a>`justfile`: the recipe `check`. 6 lines added. [The file](https://github.com/rahuldave/cheesecracker/blob/e89e0cfa991257757c887f687087a9c77c093e20/justfile).
  - look at `just fmt --check`

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

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/e89e0cfa991257757c887f687087a9c77c093e20). [The code after it](https://github.com/rahuldave/cheesecracker/tree/step-05).

This part ends at the code at `step-05`. [The code after this part](https://github.com/rahuldave/cheesecracker/tree/step-05)

<a id="step-06.3"></a>

## 6.3 just hook

This walk goes through [rahuldave/cheesecracker](https://github.com/rahuldave/cheesecracker) on GitHub. This part starts from the code at `009d77653ead`, partway through `step-06`. [The code before this part](https://github.com/rahuldave/cheesecracker/tree/009d77653eadfde09e438656cde77b5eac489115)

This part does not start where the part before it ends, because the walk skips some changes. This command puts the files that this part starts from in place of yours. Run it, and then go on:

```bash
git restore --source e89e0cfa9912 --staged -- :/ && git restore --source 009d77653ead --staged --worktree -- :/
```

```bash
just setup
```

The whole tutorial then adds pre-commit, a tool that runs checks before every commit. git runs them through a hook, a
file in git's own folder. `just setup` installs only Python and the packages, so the hook gets a recipe of its own.

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
git restore --source 009d77653ead --staged -- :/ && git restore --source 57ec2e24ecd4 --staged --worktree -- :/
```

#### What changed

- <a id="step-06.3-justfile-424f0b"></a>`justfile`: the recipe `hook`. 4 lines added. [The file](https://github.com/rahuldave/cheesecracker/blob/57ec2e24ecd4946823edbfad27bc2296f80e3847/justfile).
  - look at `uv run pre-commit install`

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

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/57ec2e24ecd4946823edbfad27bc2296f80e3847). [The code after it](https://github.com/rahuldave/cheesecracker/tree/step-06).

This part ends at the code at `step-06`. [The code after this part](https://github.com/rahuldave/cheesecracker/tree/step-06)

<a id="step-07.3"></a>

## 7.3 The Docker recipes

This walk goes through [rahuldave/cheesecracker](https://github.com/rahuldave/cheesecracker) on GitHub. This part starts from the code at `0acb4556f724`, partway through `step-07`. [The code before this part](https://github.com/rahuldave/cheesecracker/tree/0acb4556f724cd94e4a71ab4fff739a974413fc0)

This part does not start where the part before it ends, because the walk skips some changes. This command puts the files that this part starts from in place of yours. Run it, and then go on:

```bash
git restore --source 57ec2e24ecd4 --staged -- :/ && git restore --source 0acb4556f724 --staged --worktree -- :/
```

```bash
just setup
```

The whole tutorial then writes `.dockerignore`, and a `Dockerfile` with the run image. Three recipes build an image,
start its server and stop it.

### 7.3 just docker-build, just docker-serve and just docker-stop

Three recipes give the Docker commands short names. `docker-build` builds an image of the `Dockerfile`.
`docker-serve` starts the server of the run image in the background, at port 9000, and `docker-stop` stops it.

Apply the patch to [`justfile`](#step-07.3-justfile-424f0b), from the top of the project:

```bash
curl -fsSL https://github.com/rahuldave/cheesecracker/commit/d444e0387d256c62024581e98f59d670cbc93296.diff | git apply --include=justfile
```

Read the recipes `docker-build`, `docker-serve` and `docker-stop` at the end of the `justfile`.

```bash
just docker-build run
```

Docker takes each layer that has not changed since the last build from its cache, and prints `CACHED` for it.

```bash
just docker-serve
```

The recipe prints `docker-serve: http://localhost:9000, in the container cheesecracker-run`. Ask the server for its
health.

```bash
curl -s localhost:9000/health
```

The server in Docker answers `{"status":"healthy"}`, as the server on this machine did. The container runs in the
background, so you do not see what it prints. Docker keeps that output, and `docker logs` shows it.

```bash
docker logs cheesecracker-run
```

uvicorn printed `Uvicorn running on http://0.0.0.0:9000`, and started four server processes, its workers.

```bash
just docker-stop
```

Docker prints the name of the container, `cheesecracker-run`, and the server stops.

Check your files against this move. The command prints nothing when they match, and otherwise shows what differs:

```bash
git check d444e0387d25
```

If you are stuck, this command puts the files of this move in place of yours. A file that you made yourself stays:

```bash
git restore --source 0acb4556f724 --staged -- :/ && git restore --source d444e0387d25 --staged --worktree -- :/
```

#### What changed

- <a id="step-07.3-justfile-424f0b"></a>`justfile`: the recipes `docker-build`, `docker-serve` and `docker-stop`. 20 lines added. [The file](https://github.com/rahuldave/cheesecracker/blob/d444e0387d256c62024581e98f59d670cbc93296/justfile).
  - look at `docker run --rm --detach --name cheesecracker-run --publish 9000:9000 cheesecracker-run > /dev/null || exit 1`

<details open><summary>The patch to <code>justfile</code></summary>

```diff
@@ -62,3 +62,23 @@ check:
     just fmt --check
     just lint
     just test -q
+
+# Build an image of the Dockerfile: just docker-build run
+docker-build kind:
+    docker build --target "$1" --tag "cheesecracker-$1" .
+
+# Start the server in the run image, in the background, at http://localhost:9000, as it runs in production. just docker-stop stops it
+docker-serve:
+    #!/usr/bin/env sh
+    if curl -sf localhost:9000/health > /dev/null; then echo "docker-serve: a server already answers at http://localhost:9000" >&2; exit 1; fi
+    docker run --rm --detach --name cheesecracker-run --publish 9000:9000 cheesecracker-run > /dev/null || exit 1
+    for i in $(seq 1 30); do
+        if curl -sf localhost:9000/health > /dev/null; then echo "docker-serve: http://localhost:9000, in the container cheesecracker-run"; exit 0; fi
+        sleep 1
+    done
+    echo "docker-serve: no answer after 30 seconds; read docker logs cheesecracker-run" >&2
+    exit 1
+
+# Stop the server of just docker-serve, and remove its container
+docker-stop:
+    docker stop cheesecracker-run
```

</details>

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/d444e0387d256c62024581e98f59d670cbc93296). [The code after it](https://github.com/rahuldave/cheesecracker/tree/step-07).

This part ends at the code at `step-07`. [The code after this part](https://github.com/rahuldave/cheesecracker/tree/step-07)

<a id="step-08.4"></a>

## 8.4 just test-system

This walk goes through [rahuldave/cheesecracker](https://github.com/rahuldave/cheesecracker) on GitHub. This part starts from the code at `19813cf32d08`, partway through `step-08`. [The code before this part](https://github.com/rahuldave/cheesecracker/tree/19813cf32d0843eea486048243e1eaf62b16a0fe)

This part does not start where the part before it ends, because the walk skips some changes. This command puts the files that this part starts from in place of yours. Run it, and then go on:

```bash
git restore --source d444e0387d25 --staged -- :/ && git restore --source 19813cf32d08 --staged --worktree -- :/
```

```bash
just setup
just docker-build run
just docker-serve
```

The whole tutorial then adds the package requests, and writes system tests. A system test sends real HTTP requests to
the server at port 9000 of this machine. The system tests need a running server, so `just test` leaves them out, and
`just docker-serve` above starts one. The system tests get a recipe of their own, apart from `just check`.

### 8.4 just test-system runs the system tests

The system tests have a recipe of their own, `test-system`, apart from `just test` and `just check`. A missing server
fails them, and never stops a commit.

Apply the patch to [`justfile`](#step-08.4-justfile-424f0b), from the top of the project:

```bash
curl -fsSL https://github.com/rahuldave/cheesecracker/commit/277d54eebce4a1727ed3d8a5c46570ca4a95f75d.diff | git apply --include=justfile
```

Read the recipe `test-system` at the end of the `justfile`.

```bash
just test-system -q
```

pytest prints `5 passed`. Then stop the server, and run the system tests again.

```bash
just docker-stop
just test-system -q
```

pytest fails, with `1 passed, 4 errors`. Each error says `no server at http://localhost:9000: start one with just
docker-serve`.

Check your files against this move. The command prints nothing when they match, and otherwise shows what differs:

```bash
git check 277d54eebce4
```

If you are stuck, this command puts the files of this move in place of yours. A file that you made yourself stays:

```bash
git restore --source 19813cf32d08 --staged -- :/ && git restore --source 277d54eebce4 --staged --worktree -- :/
```

#### What changed

- <a id="step-08.4-justfile-424f0b"></a>`justfile`: the recipe `test-system`. 4 lines added. [The file](https://github.com/rahuldave/cheesecracker/blob/277d54eebce4a1727ed3d8a5c46570ca4a95f75d/justfile).
  - look at `uv run pytest tests/system "$@"`

<details open><summary>The patch to <code>justfile</code></summary>

```diff
@@ -82,3 +82,7 @@ docker-serve:
 # Stop the server of just docker-serve, and remove its container
 docker-stop:
     docker stop cheesecracker-run
+
+# Run the system tests against the server at http://localhost:9000. They fail when no server answers there
+test-system *args:
+    uv run pytest tests/system "$@"
```

</details>

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/277d54eebce4a1727ed3d8a5c46570ca4a95f75d). [The code after it](https://github.com/rahuldave/cheesecracker/tree/step-08).

This part ends at the code at `step-08`. [The code after this part](https://github.com/rahuldave/cheesecracker/tree/step-08)

<a id="step-09.2"></a>

## 9.2 just docker-shell and just docker-test

This walk goes through [rahuldave/cheesecracker](https://github.com/rahuldave/cheesecracker) on GitHub. This part starts from the code at `ccf62d330f28`, partway through `step-09`. [The code before this part](https://github.com/rahuldave/cheesecracker/tree/ccf62d330f28c6eec4c0b993e1a71e2c11a8b5af)

This part does not start where the part before it ends, because the walk skips some changes. This command puts the files that this part starts from in place of yours. Run it, and then go on:

```bash
git restore --source 277d54eebce4 --staged -- :/ && git restore --source ccf62d330f28 --staged --worktree -- :/
```

```bash
just setup
just docker-build dev
```

The whole tutorial then adds the dev image to the `Dockerfile`. Two recipes run it, with the folder of the project
mounted inside.

### 9.2 just docker-shell and just docker-test

Two recipes run the dev image with this folder mounted at `/work`. An edit here shows in there at once, so the image
needs no rebuild after a change of the code. `docker-shell` opens a shell in the image. `docker-test` runs the recipe
that you name in the image, or `just check` if you name none.

Apply the patch to [`justfile`](#step-09.2-justfile-424f0b), from the top of the project:

```bash
curl -fsSL https://github.com/rahuldave/cheesecracker/commit/0701e9244ab2384538eda4d623e1d047ff3f8258.diff | git apply --include=justfile
```

Read the recipes `docker-shell` and `docker-test` at the end of the `justfile`.

```bash
just docker-test
```

`just check` runs in Linux. The last line is `21 passed`. The system tests do not run, because `just check` leaves them
out.

```bash
just docker-test test tests/unit -q
```

The eleven unit tests pass in Linux.

Check your files against this move. The command prints nothing when they match, and otherwise shows what differs:

```bash
git check 0701e9244ab2
```

If you are stuck, this command puts the files of this move in place of yours. A file that you made yourself stays:

```bash
git restore --source ccf62d330f28 --staged -- :/ && git restore --source 0701e9244ab2 --staged --worktree -- :/
```

#### What changed

- <a id="step-09.2-justfile-424f0b"></a>`justfile`: the recipes `docker-shell` and `docker-test`. 8 lines added. [The file](https://github.com/rahuldave/cheesecracker/blob/0701e9244ab2384538eda4d623e1d047ff3f8258/justfile).
  - look at `docker run --rm -v "$PWD":/work cheesecracker-dev just "${@:-check}"`

<details open><summary>The patch to <code>justfile</code></summary>

```diff
@@ -86,3 +86,11 @@ docker-stop:
 # Run the system tests against the server at http://localhost:9000. They fail when no server answers there
 test-system *args:
     uv run pytest tests/system "$@"
+
+# A shell in the dev image, with this folder mounted at /work: an edit here shows in there at once
+docker-shell:
+    docker run --rm -it -v "$PWD":/work cheesecracker-dev
+
+# Run a recipe in the dev image, with this folder mounted at /work. Alone, just check: just docker-test test tests/unit
+docker-test *args:
+    docker run --rm -v "$PWD":/work cheesecracker-dev just "${@:-check}"
```

</details>

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/0701e9244ab2384538eda4d623e1d047ff3f8258). [The code after it](https://github.com/rahuldave/cheesecracker/tree/step-09).

This part ends at the code at `step-09`. [The code after this part](https://github.com/rahuldave/cheesecracker/tree/step-09)
