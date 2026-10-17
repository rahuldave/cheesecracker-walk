<!-- Made by the walk skill's render.py from the facts of the walk. Do not edit it by hand: run just walk-update again. -->

# The tests of cheesecracker

**Read the tutorial.** Each change explained, with the code to look at. [Build it yourself](build.md) is the other way to read it.

Each part ends with a section called **What changed**. It lists every file that the part changes, with a link to the file, and then the whole diff of each file, folded. A lockfile and a binary file have a link alone. The text above it explains the change.

We build the tests of cheesecracker, the API server of the Cheese App. A test runs the code and checks what it gives
back, so a later change cannot break the server without a failure. [pytest](https://docs.pytest.org/) runs the tests. It finds the
test functions of a project, runs them, and reports each failure. This tutorial takes the parts about tests out of
the whole tutorial of cheesecracker. It leaves out ruff, pre-commit and the dev image.

We make the project in moves. A move is one small change, made by one patch or by one command. You make each move,
and the page checks your files against it.

The tests come in three kinds, from the bottom of the testing pyramid to its top:

| Kind | What it calls | What it needs |
|---|---|---|
| Unit | One function alone, `power()` | Nothing |
| Integration | The app, through FastAPI's `TestClient`, in the same process | No server |
| System | The running server, over HTTP | A server at port 9000, the run image in Docker |

Some changes of the whole tutorial are not in this one. After such a gap, the next section says what those changes
did, and starts from the code that they left. It first runs `just setup`, which installs the packages of that code.

<a id="step-00"></a>

## 0 The project

This walk goes through [rahuldave/cheesecracker](https://github.com/rahuldave/cheesecracker) on GitHub. This part is where the project starts, at `step-00`. [The code of this part](https://github.com/rahuldave/cheesecracker/tree/step-00)

The project starts empty, with a `pyproject.toml`, a lockfile and a `justfile`. The next sections give it code and
tests.

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

A unit test calls one function alone. The first function of the server, `power()`, gets eleven of them.

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

[± See the new file `src/api-service/api/__init__.py`](#step-01.1-src-api-service-api-__init__.py-e2e18a-diff)

[± See the new file `src/api-service/api/utils.py`](#step-01.1-src-api-service-api-utils.py-18f1f8-diff)

The new file `src/api-service/api/utils.py` has a docstring and one function, `power(base, exponent)`. It returns
`base**exponent`. `src/api-service/api/__init__.py` has one comment, and makes `api` a package.

The package is not on the import path of the project yet. So the check runs Python in the folder `src/api-service/`,
where `api` is a folder of its own.

```bash
uv run --directory src/api-service python -c "from api.utils import power; print(power(2, 3))"
```

Python prints `8`, which is 2 to the power of 3.

#### What changed

- <a id="step-01.1-src-api-service-api-__init__.py-e2e18a"></a>`src/api-service/api/__init__.py`: one comment, which makes the folder a package. New, 1 line. [The file](https://github.com/rahuldave/cheesecracker/blob/bb1701fd54ba7c6aa9bd971c6651f34439f004d6/src/api-service/api/__init__.py).
- <a id="step-01.1-src-api-service-api-utils.py-18f1f8"></a>`src/api-service/api/utils.py`: the function `power()`. New, 9 lines. [The file](https://github.com/rahuldave/cheesecracker/blob/bb1701fd54ba7c6aa9bd971c6651f34439f004d6/src/api-service/api/utils.py).
  - look at `return base**exponent`

<a id="step-01.1-src-api-service-api-__init__.py-e2e18a-diff"></a>

<details><summary>The diff of <code>src/api-service/api/__init__.py</code></summary>

```diff
@@ -0,0 +1 @@
+# API Package
```

</details>

<a id="step-01.1-src-api-service-api-utils.py-18f1f8-diff"></a>

<details><summary>The diff of <code>src/api-service/api/utils.py</code></summary>

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

This change came from running `uv add --group test pytest`.

The command adds pytest to the group `test` of `pyproject.toml`. It writes the exact versions of pytest and of the
packages that it needs into `uv.lock`.

```bash
uv run pytest --version
```

uv prints the version of pytest, `pytest 9.1.1` on the day of the class.

#### What changed

- <a id="step-01.2-pyproject.toml-5d07e7"></a>`pyproject.toml`: pytest, in the group `test`. 5 lines added. It comes from `uv add --group test pytest`. [The file](https://github.com/rahuldave/cheesecracker/blob/fc8c3bce83604277393460c4eb243d176d51905c/pyproject.toml).
  - look at `"pytest>=9.1.1",`
- <a id="step-01.2-uv.lock-8356d8"></a>`uv.lock`: the exact version of pytest, and of the packages that it needs. 71 lines added. It comes from `uv add --group test pytest`. [The file](https://github.com/rahuldave/cheesecracker/blob/fc8c3bce83604277393460c4eb243d176d51905c/uv.lock).

<a id="step-01.2-pyproject.toml-5d07e7-diff"></a>

<details><summary>The diff of <code>pyproject.toml</code></summary>

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

[± See the change to `pyproject.toml`](#step-01.3-pyproject.toml-5d07e7-diff)

The new table `[tool.pytest.ini_options]` of `pyproject.toml` has two settings. `testpaths` makes pytest
look for tests in `tests/` only. `pythonpath` puts `src/api-service/` on the import path, so a test can import `api`.

```bash
uv run pytest
```

pytest collects 0 items, and fails with the code 5, because there are no tests yet. It also warns that it found no
files in `tests/`, which does not exist yet.

#### What changed

- <a id="step-01.3-pyproject.toml-5d07e7"></a>`pyproject.toml`: the table `[tool.pytest.ini_options]`, with the folder of the tests and the folder of the package. 5 lines added. [The file](https://github.com/rahuldave/cheesecracker/blob/ff82ac1870a7e1937a6fb4d25ac171249378f299/pyproject.toml).
  - look at `pythonpath = ["src/api-service"]`

<a id="step-01.3-pyproject.toml-5d07e7-diff"></a>

<details><summary>The diff of <code>pyproject.toml</code></summary>

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

[± See the new file `tests/unit/test_utils.py`](#step-01.4-tests-unit-test_utils.py-3d93fe-diff)

The new file `tests/unit/test_utils.py` has the class `TestPower`, with eleven tests. Each `assert` compares a result
of `power()` with its answer. `test_power_invalid_inputs` expects a `TypeError` for a string, `None`, a list or a
dictionary.

```bash
uv run pytest -q
```

pytest prints `11 passed`. So `power()` gives the answer that each test expects.

#### What changed

- <a id="step-01.4-tests-unit-test_utils.py-3d93fe"></a>`tests/unit/test_utils.py`: eleven unit tests of `power()`. New, 91 lines. [The file](https://github.com/rahuldave/cheesecracker/blob/cbc98b162671bce7a7ba82d7d18c7dc6d530b746/tests/unit/test_utils.py).
  - look at `assert power(2, 3) == 8`

<a id="step-01.4-tests-unit-test_utils.py-3d93fe-diff"></a>

<details><summary>The diff of <code>tests/unit/test_utils.py</code></summary>

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

[± See the change to `justfile`](#step-01.5-justfile-424f0b-diff)

The recipe `test`, at the end of the `justfile`, runs `uv run pytest "$@"`. `"$@"` stands for the words after
`just test`, so `just test tests/unit` runs the unit tests alone.

```bash
just test -q
```

just prints the command of the recipe, and pytest prints `11 passed`.

#### What changed

- <a id="step-01.5-justfile-424f0b"></a>`justfile`: the recipe `test`. 4 lines added. [The file](https://github.com/rahuldave/cheesecracker/blob/e86e0c3d10bbe9409674f1c65ce4afdc38e7f76f/justfile).
  - look at `uv run pytest "$@"`

<a id="step-01.5-justfile-424f0b-diff"></a>

<details><summary>The diff of <code>justfile</code></summary>

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

## 2 The server under test

This walk goes through [rahuldave/cheesecracker](https://github.com/rahuldave/cheesecracker) on GitHub. This part starts from the code at `step-01`. [The code before this part](https://github.com/rahuldave/cheesecracker/tree/step-01)

The integration tests and the system tests need a server to call. This section writes it, with two answers, `/` and
`/health`, and the recipes `just serve` and `just stop`.

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

This change came from running `uv add fastapi uvicorn`.

The command adds both packages to `dependencies` in `pyproject.toml`, and writes their versions into `uv.lock`.

```bash
uv run uvicorn --version
```

uvicorn prints its version, and the version of Python that runs it.

#### What changed

- <a id="step-02.1-pyproject.toml-5d07e7"></a>`pyproject.toml`: FastAPI and uvicorn, the dependencies of the server. 4 lines added, 1 removed. It comes from `uv add fastapi uvicorn`. [The file](https://github.com/rahuldave/cheesecracker/blob/18dd9dfb1c9597e50a8b404a6cb6379c1c149320/pyproject.toml).
  - look at `"fastapi>=0.143.2",`
- <a id="step-02.1-uv.lock-8356d8"></a>`uv.lock`: the exact versions of FastAPI, uvicorn, and the packages that they need. 263 lines added. It comes from `uv add fastapi uvicorn`. [The file](https://github.com/rahuldave/cheesecracker/blob/18dd9dfb1c9597e50a8b404a6cb6379c1c149320/uv.lock).

<a id="step-02.1-pyproject.toml-5d07e7-diff"></a>

<details><summary>The diff of <code>pyproject.toml</code></summary>

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

[± See the new file `src/api-service/api/service.py`](#step-02.2-src-api-service-api-service.py-1e1e71-diff)

The new file `src/api-service/api/service.py` makes `app`, with a title and a version. The CORS middleware lets a web
page from any address call the server. After the comment `# Routes`, `get_index()` answers `/`, and `health_check()`
answers `/health`.

```bash
uv run --directory src/api-service python -c "from api.service import app; print([r.path for r in app.routes])"
```

Python prints the routes of the app. The last two are `/` and `/health`. FastAPI adds the first four, for its
documentation pages.

#### What changed

- <a id="step-02.2-src-api-service-api-service.py-1e1e71"></a>`src/api-service/api/service.py`: the FastAPI app, its CORS middleware, and the routes `/` and `/health`. New, 29 lines. [The file](https://github.com/rahuldave/cheesecracker/blob/acee14f7dff43d6c5b76234b3a016985d74c60d2/src/api-service/api/service.py).
  - look at `return {"status": "healthy"}`

<a id="step-02.2-src-api-service-api-service.py-1e1e71-diff"></a>

<details><summary>The diff of <code>src/api-service/api/service.py</code></summary>

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

[± See the change to `justfile`](#step-02.3-justfile-424f0b-diff)

[± See the change to `.gitignore`](#step-02.3-.gitignore-a5cc29-diff)

The recipe `serve` runs `uvicorn api.service:app` with `&`, so the server runs in the background. `--app-dir` tells
uvicorn where the package `api` is, and `--reload` restarts the server when a file of the code changes. The recipe
keeps the number of the process in `.serve.pid`, and asks `/health` once a second until the server answers. `stop`
stops that process, and waits until the port is free. `.gitignore` now lists `.serve.log` and `.serve.pid`, which git never takes.

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

#### What changed

- <a id="step-02.3-.gitignore-a5cc29"></a>`.gitignore`: the log of the server and the number of its process. 4 lines added. [The file](https://github.com/rahuldave/cheesecracker/blob/69c3bf65c78cf8e96bf8f9ea1dde29279db1351a/.gitignore).
  - look at `.serve.pid`
- <a id="step-02.3-justfile-424f0b"></a>`justfile`: the recipes `serve` and `stop`. 26 lines added. [The file](https://github.com/rahuldave/cheesecracker/blob/69c3bf65c78cf8e96bf8f9ea1dde29279db1351a/justfile).
  - look at `uv run uvicorn api.service:app --app-dir src/api-service --host 0.0.0.0 --port 9000 --reload > .serve.log 2>&1 &`

<a id="step-02.3-.gitignore-a5cc29-diff"></a>

<details><summary>The diff of <code>.gitignore</code></summary>

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

<details><summary>The diff of <code>justfile</code></summary>

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

## 3 Integration tests

This walk goes through [rahuldave/cheesecracker](https://github.com/rahuldave/cheesecracker) on GitHub. This part starts from the code at `step-02`. [The code before this part](https://github.com/rahuldave/cheesecracker/tree/step-02)

An integration test sends a request to the app, with no server and no network. First the app gets a new request
that does some work. `/euclidean_distance/` answers with the distance of a point from the origin.

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

[± See the change to `src/api-service/api/service.py`](#step-03.1-src-api-service-api-service.py-1e1e71-diff)

`service.py` now imports `math`, and `power()` from `api.utils`. The new function at the end,
`calculate_euclidean_distance()`, answers `/euclidean_distance/`. It takes `x` and `y`, with 1 and 2 as defaults,
and returns them with the distance.

Start the server, and ask it for the distance of the point (3, 4). The server runs to the end of this part, and
restarts when a file of the code changes, because of `--reload`.

```bash
just serve
curl -s "localhost:9000/euclidean_distance/?x=3&y=4"
```

The server answers `{"x":3.0,"y":4.0,"distance":5.0}`.

#### What changed

- <a id="step-03.1-src-api-service-api-service.py-1e1e71"></a>`src/api-service/api/service.py`: the imports of `math` and `power()`, and the endpoint `/euclidean_distance/`. 12 lines added. [The file](https://github.com/rahuldave/cheesecracker/blob/e576e7bb9e550b1a572a764b15f8d0bfde229d97/src/api-service/api/service.py).
  - look at `z = power(x, 2) + power(y, 2)`

<a id="step-03.1-src-api-service-api-service.py-1e1e71-diff"></a>

<details><summary>The diff of <code>src/api-service/api/service.py</code></summary>

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

This change came from running `uv add --group test httpx2 pytest-asyncio`.

The command adds both packages to the group `test`, and writes their versions into `uv.lock`.

```bash
just test -q
```

pytest still prints `11 passed`. The new packages change no test.

#### What changed

- <a id="step-03.2-pyproject.toml-5d07e7"></a>`pyproject.toml`: httpx2 and pytest-asyncio, in the group `test`. 2 lines added. It comes from `uv add --group test httpx2 pytest-asyncio`. [The file](https://github.com/rahuldave/cheesecracker/blob/3c75ffb1a089431a2455a69e9bcb3ec0302224f0/pyproject.toml).
  - look at `"httpx2>=2.13.1",`
- <a id="step-03.2-uv.lock-8356d8"></a>`uv.lock`: the exact versions of httpx2, pytest-asyncio, and the packages that they need. 68 lines added, 1 removed. It comes from `uv add --group test httpx2 pytest-asyncio`. [The file](https://github.com/rahuldave/cheesecracker/blob/3c75ffb1a089431a2455a69e9bcb3ec0302224f0/uv.lock).

<a id="step-03.2-pyproject.toml-5d07e7-diff"></a>

<details><summary>The diff of <code>pyproject.toml</code></summary>

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

[± See the new file `tests/integration/test_api.py`](#step-03.3-tests-integration-test_api.py-596674-diff)

The new file `tests/integration/test_api.py` makes `client = TestClient(app)`. The class `TestAPIEndpoints` checks
the welcome, the distance of (3, 4) and the defaults. It also checks a route that does not exist (404), and a method
that is not allowed (405). The class `TestCORS` checks the CORS headers.

```bash
just test -q
```

pytest prints `21 passed`, the eleven unit tests and the ten integration tests. The tests needed no server, so we
stop the one of `just serve`.

```bash
just stop
```

#### What changed

- <a id="step-03.3-tests-integration-test_api.py-596674"></a>`tests/integration/test_api.py`: ten integration tests, through FastAPI's `TestClient`. New, 89 lines. [The file](https://github.com/rahuldave/cheesecracker/blob/5862780b44d2f4ec87088c971dec57f23c11139e/tests/integration/test_api.py).
  - look at `response = client.get("/euclidean_distance/?x=3&y=4")`

<a id="step-03.3-tests-integration-test_api.py-596674-diff"></a>

<details><summary>The diff of <code>tests/integration/test_api.py</code></summary>

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

<a id="step-04"></a>

## 4 Coverage

This walk goes through [rahuldave/cheesecracker](https://github.com/rahuldave/cheesecracker) on GitHub. This part starts from the code at `step-03`. [The code before this part](https://github.com/rahuldave/cheesecracker/tree/step-03)

The tests pass, but nothing says which lines of the code they run. Coverage counts them.

```bash
just setup
```

Twenty-one tests pass, and we still do not know which lines of the app they run. Coverage counts the lines of the
package `api` that run during the tests, and lists the lines that no test ran.

<a id="step-04.1"></a>

### 4.1 pytest-cov, to measure what the tests run

A line that no test runs can break with no test failing. [pytest-cov](https://pytest-cov.readthedocs.io/) adds the
option `--cov` to pytest. The option prints a table with each file and the share of its lines that ran. It is a tool
of the tests, so it goes into the group `test`.

This change came from running `uv add --group test pytest-cov`.

The check runs the unit tests alone, with coverage of the package `api`.

```bash
uv run pytest tests/unit --cov=api -q
```

The table shows `utils.py` at 100% and `service.py` at 0%. The unit tests call `power()`, and never the app.

#### What changed

- <a id="step-04.1-pyproject.toml-5d07e7"></a>`pyproject.toml`: pytest-cov, in the group `test`. 1 line added. It comes from `uv add --group test pytest-cov`. [The file](https://github.com/rahuldave/cheesecracker/blob/54de20e8055ec62726d8956cac3a944433dc404c/pyproject.toml).
  - look at `"pytest-cov>=7.1.0",`
- <a id="step-04.1-uv.lock-8356d8"></a>`uv.lock`: the exact versions of pytest-cov, and of coverage, which it uses. 115 lines added. It comes from `uv add --group test pytest-cov`. [The file](https://github.com/rahuldave/cheesecracker/blob/54de20e8055ec62726d8956cac3a944433dc404c/uv.lock).

<a id="step-04.1-pyproject.toml-5d07e7-diff"></a>

<details><summary>The diff of <code>pyproject.toml</code></summary>

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

<a id="step-04.2"></a>

### 4.2 just cov shows which lines the tests run

The recipe `cov` runs every test with coverage, and adds the column `Missing`, with the numbers of the lines that no
test ran.

[± See the change to `justfile`](#step-04.2-justfile-424f0b-diff)

The recipe `cov`, at the end of the `justfile`, runs pytest with `--cov=api --cov-report=term-missing`, and passes
on to pytest any words that you type after `just cov`.

```bash
just cov -q
```

The total is 95%. `service.py` misses line 31, the answer of `/health`, because no test asks for it.

#### What changed

- <a id="step-04.2-justfile-424f0b"></a>`justfile`: the recipe `cov`. 4 lines added. [The file](https://github.com/rahuldave/cheesecracker/blob/bfb9c783c67c31254ce8136784738c0f2fc17fb5/justfile).
  - look at `uv run pytest --cov=api --cov-report=term-missing "$@"`

<a id="step-04.2-justfile-424f0b-diff"></a>

<details><summary>The diff of <code>justfile</code></summary>

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

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/bfb9c783c67c31254ce8136784738c0f2fc17fb5).

This part ends at the code at `step-04`. [The code after this part](https://github.com/rahuldave/cheesecracker/tree/step-04)

<a id="step-07"></a>

## 7 The run image, a server to test against

This walk goes through [rahuldave/cheesecracker](https://github.com/rahuldave/cheesecracker) on GitHub. This part starts from the code at `step-06`. [The code before this part](https://github.com/rahuldave/cheesecracker/tree/step-06)

The whole tutorial then makes a change in a hurry. It adds an import that nothing uses, a function
`unused_function()` that nothing calls, and a long `message` in the answer of `/euclidean_distance/`. Then ruff, a
tool that formats code and checks it, fixes the spaces and blank lines, and removes the import. A person shortens the
message. pre-commit then runs the checks before each commit. The tests pass all along.

A system test needs a server that runs as it will run in production. Docker builds that server as an image, the run
image.

```bash
just setup
```

The server runs on this machine, with its `.venv` and this version of macOS or Linux. Production is another machine,
such as a server on Google Cloud. A Docker image holds the server with everything it needs, so it runs the same on
any machine with Docker. The run image is the image that production runs, our deployment.

<a id="step-07.1"></a>

### 7.1 what docker build leaves out

[Docker](https://docs.docker.com/) runs programs in containers. A container is a process with its own files, cut off
from the rest of the machine. A `Dockerfile` lists the instructions that build an image. It starts from a base system, such
as Debian with Python, and adds the files and the commands. `docker build` makes the image, and `docker run` starts a
container from it. On macOS and Windows, the app Docker Desktop runs Docker.

`docker build` sends the files of the folder to Docker. The folder has `.venv`, the caches of the tools, git and maybe
secrets. None of them belongs in an image. `.dockerignore` lists them.

[± See the new file `.dockerignore`](#step-07.1-.dockerignore-05a721-diff)

The new file `.dockerignore` lists `.venv/`, the caches of pytest, ruff and coverage, `.git/`, `secrets/` and `.env`.

```bash
cat .dockerignore
```

`cat` prints the list.

#### What changed

- <a id="step-07.1-.dockerignore-05a721"></a>`.dockerignore`: what `docker build` never sends to Docker. New, 10 lines. [The file](https://github.com/rahuldave/cheesecracker/blob/2a28a5c704d6d9ed8200aeb8962aa01e5c2ac552/.dockerignore).
  - look at `.venv/`

<a id="step-07.1-.dockerignore-05a721-diff"></a>

<details><summary>The diff of <code>.dockerignore</code></summary>

```diff
@@ -0,0 +1,10 @@
+# What docker build never sends to Docker: the environment, the caches, git and the secrets
+.venv/
+__pycache__/
+.pytest_cache/
+.ruff_cache/
+.coverage
+htmlcov/
+.git/
+secrets/
+.env
```

</details>

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/2a28a5c704d6d9ed8200aeb8962aa01e5c2ac552). [The code after it](https://github.com/rahuldave/cheesecracker/tree/2a28a5c704d6d9ed8200aeb8962aa01e5c2ac552).

<a id="step-07.2"></a>

### 7.2 the run image, the server as it runs in production

The run image needs Python, the dependencies of the server, and the code. It does not need uv, the tests, ruff or
pre-commit. The `Dockerfile` builds the image in stages, and the run image takes only what the server needs.

[± See the new file `Dockerfile`](#step-07.2-Dockerfile-6651dd-diff)

The stage `uv` takes uv from its own image. The stage `build` installs the dependencies of `uv.lock` into
`/opt/venv`, with `--no-default-groups`, so no test tool and no ruff. The stage `run` copies `/opt/venv` and the
package `api`, and runs uvicorn as the user `app`, with four workers.

```bash
docker build --target run --tag cheesecracker-run .
```

Docker prints each line of the `Dockerfile` as it runs. Near the end, the line `naming to
docker.io/library/cheesecracker-run` names the image.

```bash
docker image ls cheesecracker-run
```

The image is about 173 MB, nearly all of it Python and the dependencies of the server.

#### What changed

- <a id="step-07.2-Dockerfile-6651dd"></a>`Dockerfile`: the stages `uv`, `build` and `run`. New, 34 lines. [The file](https://github.com/rahuldave/cheesecracker/blob/0acb4556f724cd94e4a71ab4fff739a974413fc0/Dockerfile).
  - look at `RUN uv sync --locked --no-default-groups`

<a id="step-07.2-Dockerfile-6651dd-diff"></a>

<details><summary>The diff of <code>Dockerfile</code></summary>

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

<a id="step-07.3"></a>

### 7.3 just docker-build, just docker-serve and just docker-stop

Three recipes give the Docker commands short names. `docker-build` builds an image of the `Dockerfile`.
`docker-serve` starts the server of the run image in the background, at port 9000, and `docker-stop` stops it.

[± See the change to `justfile`](#step-07.3-justfile-424f0b-diff)

The recipe `docker-build` takes the name of a stage, and runs `docker build --target` with it. `docker-serve` runs the
image `cheesecracker-run` with `--detach`, so the container runs in the background, as in the original tutorial's
`docker run -d`. `--publish` connects its port 9000 to port 9000 of this machine. The recipe then asks `/health`
until the server answers. `docker-stop` stops the container, and `--rm` removes it.

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

#### What changed

- <a id="step-07.3-justfile-424f0b"></a>`justfile`: the recipes `docker-build`, `docker-serve` and `docker-stop`. 20 lines added. [The file](https://github.com/rahuldave/cheesecracker/blob/d444e0387d256c62024581e98f59d670cbc93296/justfile).
  - look at `docker run --rm --detach --name cheesecracker-run --publish 9000:9000 cheesecracker-run > /dev/null || exit 1`

<a id="step-07.3-justfile-424f0b-diff"></a>

<details><summary>The diff of <code>justfile</code></summary>

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

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/d444e0387d256c62024581e98f59d670cbc93296).

This part ends at the code at `step-07`. [The code after this part](https://github.com/rahuldave/cheesecracker/tree/step-07)

<a id="step-08"></a>

## 8 System tests, against the run image

This walk goes through [rahuldave/cheesecracker](https://github.com/rahuldave/cheesecracker) on GitHub. This part starts from the code at `step-07`. [The code before this part](https://github.com/rahuldave/cheesecracker/tree/step-07)

The system tests send real HTTP requests to the run image. They need a server, so they run apart from the other
tests, and fail when no server answers.

```bash
just setup
```

The integration tests call the app in the same process as the tests. They cannot tell whether the image starts, or
whether the server answers on its port. A system test sends real HTTP requests to the running server, as the Cheese
App does. Here the server is the run image, our deployment, which `just docker-serve` starts.

<a id="step-08.1"></a>

### 8.1 requests, for tests that send real HTTP requests

The system tests send their requests with the package [requests](https://requests.readthedocs.io/), the most used
HTTP library of Python. It is a tool of the tests, so it goes into the group
`test`.

This change came from running `uv add --group test requests`.

```bash
uv run python -c "import requests; print(requests.__version__)"
```

Python prints the version of requests.

#### What changed

- <a id="step-08.1-pyproject.toml-5d07e7"></a>`pyproject.toml`: requests, in the group `test`. 1 line added. It comes from `uv add --group test requests`. [The file](https://github.com/rahuldave/cheesecracker/blob/de0780b72a5cad4f0ed70e9c0cdcdee9d1028562/pyproject.toml).
  - look at `"requests>=2.34.2",`
- <a id="step-08.1-uv.lock-8356d8"></a>`uv.lock`: the exact versions of requests, and of the packages that it needs. 166 lines added. It comes from `uv add --group test requests`. [The file](https://github.com/rahuldave/cheesecracker/blob/de0780b72a5cad4f0ed70e9c0cdcdee9d1028562/uv.lock).

<a id="step-08.1-pyproject.toml-5d07e7-diff"></a>

<details><summary>The diff of <code>pyproject.toml</code></summary>

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

<a id="step-08.2"></a>

### 8.2 the tests that run by default: unit and integration

The system tests need a running server, and the other tests do not. `just test`, `just check`, the git hook and the
dev image run every test of `testpaths`. They must not depend on a server, so `testpaths` names the two folders of
tests that need none. The system tests then run only when we ask for them.

[± See the change to `pyproject.toml`](#step-08.2-pyproject.toml-5d07e7-diff)

`testpaths` in `[tool.pytest.ini_options]` now names `tests/unit` and `tests/integration`, in place of `tests`. A
folder that pytest gets on its command line, such as `tests/system`, still runs.

```bash
just test -q
```

pytest prints `21 passed`, as before.

#### What changed

- <a id="step-08.2-pyproject.toml-5d07e7"></a>`pyproject.toml`: the two folders of tests that run by default. 1 line added, 1 removed. [The file](https://github.com/rahuldave/cheesecracker/blob/c8302b4cff8bd9fc480ba4f74a0c7d34c769fef6/pyproject.toml).
  - look at `testpaths = ["tests/unit", "tests/integration"]`

<a id="step-08.2-pyproject.toml-5d07e7-diff"></a>

<details><summary>The diff of <code>pyproject.toml</code></summary>

```diff
@@ -30,7 +30,7 @@ test = [
 
 [tool.pytest.ini_options]
 # The tests are in tests/, and import the package api from src/api-service/
-testpaths = ["tests"]
+testpaths = ["tests/unit", "tests/integration"]
 pythonpath = ["src/api-service"]
 
 [tool.ruff]
```

</details>

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/c8302b4cff8bd9fc480ba4f74a0c7d34c769fef6). [The code after it](https://github.com/rahuldave/cheesecracker/tree/c8302b4cff8bd9fc480ba4f74a0c7d34c769fef6).

<a id="step-08.3"></a>

### 8.3 system tests, against the server in Docker

Five system tests send requests to `localhost:9000`. Four of them need a server there, and the fifth imports the app.
A test that needs a server must fail when none answers, as a test fails without its data. A fixture does that.

[± See the new file `tests/system/test_system_api.py`](#step-08.3-tests-system-test_system_api.py-780a11-diff)

The new file `tests/system/test_system_api.py` sets `API_BASE_URL` to `http://localhost:9000`. The fixture `server`
asks for `/health`, and fails with "no server at http://localhost:9000" when nothing answers. The four tests of
`TestAPIEndpoints` use it. `TestAPIWithoutServer` imports the app, and needs no server.

Build the run image, start its server, and run the system tests against it. The server runs to the end of this part.

```bash
just docker-build run
just docker-serve
uv run pytest tests/system -v
```

pytest prints `5 passed`. The four tests of `TestAPIEndpoints` sent their requests to the server in Docker.

```bash
just test -q
```

`just test` still prints `21 passed`, because `testpaths` leaves out `tests/system`.

#### What changed

- <a id="step-08.3-tests-system-test_system_api.py-780a11"></a>`tests/system/test_system_api.py`: five system tests, through real HTTP requests to `localhost:9000`. New, 69 lines. [The file](https://github.com/rahuldave/cheesecracker/blob/19813cf32d0843eea486048243e1eaf62b16a0fe/tests/system/test_system_api.py).
  - look at `pytest.fail(f"no server at {API_BASE_URL}: start one with just docker-serve", pytrace=False)`

<a id="step-08.3-tests-system-test_system_api.py-780a11-diff"></a>

<details><summary>The diff of <code>tests/system/test_system_api.py</code></summary>

```diff
@@ -0,0 +1,69 @@
+"""
+System tests of the API server
+Send real HTTP requests to the server at localhost:9000, the deployment
+"""
+
+import pytest
+import requests
+
+
+# Base URL for the API (assumes API is running)
+API_BASE_URL = "http://localhost:9000"
+
+
+@pytest.fixture(scope="module")
+def server():
+    """Fail the tests that use it when no server answers at localhost:9000"""
+    try:
+        requests.get(f"{API_BASE_URL}/health", timeout=2)
+    except requests.RequestException:
+        pytest.fail(f"no server at {API_BASE_URL}: start one with just docker-serve", pytrace=False)
+
+
+@pytest.mark.usefixtures("server")
+class TestAPIEndpoints:
+    """System tests of the endpoints, through the running server"""
+
+    def test_root_endpoint(self):
+        """Test the root endpoint returns welcome message"""
+        response = requests.get(f"{API_BASE_URL}/")
+        assert response.status_code == 200
+        data = response.json()
+        assert "message" in data
+        assert "Cheese App" in data["message"]
+
+    def test_health_check(self):
+        """Test health check endpoint"""
+        response = requests.get(f"{API_BASE_URL}/health")
+        assert response.status_code == 200
+        data = response.json()
+        assert data["status"] == "healthy"
+
+    def test_euclidean_distance_endpoint_default(self):
+        """Test euclidean_distance endpoint with default values"""
+        response = requests.get(f"{API_BASE_URL}/euclidean_distance/")
+        assert response.status_code == 200
+        data = response.json()
+        assert "distance" in data
+        assert data["x"] == 1.0
+        assert data["y"] == 2.0
+        assert data["distance"] == pytest.approx(2.236, rel=0.01)
+
+    def test_euclidean_distance_endpoint_custom_values(self):
+        """Test euclidean_distance endpoint with custom values (3, 4)"""
+        response = requests.get(f"{API_BASE_URL}/euclidean_distance/?x=3&y=4")
+        assert response.status_code == 200
+        data = response.json()
+        assert data["distance"] == pytest.approx(5.0)
+
+
+# Standalone tests that don't require running API
+class TestAPIWithoutServer:
+    """Tests that can run without a live server"""
+
+    def test_api_structure(self):
+        """Test that we can import the API module"""
+        from api.service import app
+
+        assert app.title == "Cheese API Server"
+        assert app.version == "v1"
```

</details>

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/19813cf32d0843eea486048243e1eaf62b16a0fe). [The code after it](https://github.com/rahuldave/cheesecracker/tree/19813cf32d0843eea486048243e1eaf62b16a0fe).

<a id="step-08.4"></a>

### 8.4 just test-system runs the system tests

The system tests have a recipe of their own, `test-system`, apart from `just test` and `just check`. A missing server
fails them, and never stops a commit.

[± See the change to `justfile`](#step-08.4-justfile-424f0b-diff)

The recipe `test-system`, at the end of the `justfile`, runs `uv run pytest tests/system`, and passes on to pytest any
words that you type after it.

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

#### What changed

- <a id="step-08.4-justfile-424f0b"></a>`justfile`: the recipe `test-system`. 4 lines added. [The file](https://github.com/rahuldave/cheesecracker/blob/277d54eebce4a1727ed3d8a5c46570ca4a95f75d/justfile).
  - look at `uv run pytest tests/system "$@"`

<a id="step-08.4-justfile-424f0b-diff"></a>

<details><summary>The diff of <code>justfile</code></summary>

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

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/277d54eebce4a1727ed3d8a5c46570ca4a95f75d).

This part ends at the code at `step-08`. [The code after this part](https://github.com/rahuldave/cheesecracker/tree/step-08)
