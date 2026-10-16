<!-- Made by the walk skill's render.py from the facts of the walk. Do not edit it by hand: run just walk-update again. -->

# cheesecracker, one change at a time

**Read the tutorial.** Each change explained, with the code to look at. [Build it yourself](build.md) is the other way to read it.

Each part ends with a section called **What changed**. It lists every file that the part changes, with a link to the file, and then the whole diff of each file, folded. A lockfile and a binary file have a link alone. The text above it explains the change.

cheesecracker is the API server of the Cheese App. It is a program that waits for requests from other programs
over HTTP, the protocol of the web, and sends back answers. It answers a few kinds of request, and it is small on purpose.

The code is not the lesson. The lesson is how to test a server, check its code, and run it in Docker. Docker runs a program in a
container, with its own files and its own Python. The program runs there as in production, where real users reach it.

We build the project from an empty folder, in this order:

1. A function, `power()`, and its unit tests, which call the function alone.
2. A server that uses it, built with FastAPI, a Python library for web servers, and integration tests that call the server with no network.
3. Coverage, which shows the lines that no test runs.
4. ruff, which formats the code and checks it, and pre-commit, which runs the checks each time you save a
   version of the project with `git commit`.
5. The run image, the package of the server that Docker runs, as it runs in production. An image is the files
   that Docker starts a container from.
6. System tests, which send real HTTP requests to the server in the run image.
7. The dev image, which holds the project with every tool. It runs the checks in Linux, the system of the
   servers in production.

Everything except the Docker images runs on this machine. The tool uv installs Python and the libraries of the
project. The tool just runs each task of the project by a short name, and `curl` sends a request to a server from the
terminal. Install [uv](https://docs.astral.sh/uv/), [just](https://just.systems/) and `curl`, and
[Docker](https://docs.docker.com/get-started/get-docker/) for the last parts.

The project grows by moves. A move is one small change, made by one command, or by one patch. A patch lists the lines to add to a file and the
lines to remove. You make each move, and the
page checks your files against it. Each move ends with a command that shows what the change did.

<a id="step-00"></a>

## 0 Where we start

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

<a id="step-01"></a>

## 1 power(), and its unit tests

```bash
just setup
```

The server will compute distances, and it needs a function that raises a number to a power. We write the function
first, and tests that call it alone. A unit test calls one function, with no server and no network, so it runs in
milliseconds.

[The code before this part](https://github.com/rahuldave/cheesecracker/tree/cf50a6ef5045e08ca727aeaffdb710fad2cae096). [The code after it](https://github.com/rahuldave/cheesecracker/tree/e86e0c3d10bbe9409674f1c65ce4afdc38e7f76f).

<a id="step-01.1"></a>

### 1.1 power(), the first function of the API

The code of the API is a Python package, `api`, in the folder `src/api-service/`. Its first function, `power()`,
returns its first argument raised to the power of its second.

A patch to [`src/api-service/api/__init__.py`](#step-01.1-src-api-service-api-__init__.py-e2e18a).

A patch to [`src/api-service/api/utils.py`](#step-01.1-src-api-service-api-utils.py-18f1f8).

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

<details><summary>The diff of <code>src/api-service/api/__init__.py</code></summary>

```diff
@@ -0,0 +1 @@
+# API Package
```

</details>

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

A test calls `power()` and compares the result with the answer that we expect. pytest finds the tests of a project,
runs them, and reports each failure. The tests need pytest, but the server does not. So pytest goes into a group of
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

A patch to [`pyproject.toml`](#step-01.3-pyproject.toml-5d07e7).

The new table `[tool.pytest.ini_options]` at the end of `pyproject.toml` has two settings. `testpaths` makes pytest
look for tests in `tests/` only. `pythonpath` puts `src/api-service/` on the import path, so a test can import `api`.

```bash
uv run pytest
```

pytest collects 0 items, and fails with the code 5, because there are no tests yet. It also warns that it found no
files in `tests/`, which does not exist yet.

#### What changed

- <a id="step-01.3-pyproject.toml-5d07e7"></a>`pyproject.toml`: the table `[tool.pytest.ini_options]`, with the folder of the tests and the folder of the package. 5 lines added. [The file](https://github.com/rahuldave/cheesecracker/blob/ff82ac1870a7e1937a6fb4d25ac171249378f299/pyproject.toml).
  - look at `pythonpath = ["src/api-service"]`

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

A patch to [`tests/unit/test_utils.py`](#step-01.4-tests-unit-test_utils.py-3d93fe).

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

A patch to [`justfile`](#step-01.5-justfile-424f0b).

The recipe `test`, at the end of the `justfile`, runs `uv run pytest "$@"`. `"$@"` stands for the words after
`just test`, so `just test tests/unit` runs the unit tests alone.

```bash
just test -q
```

just prints the command of the recipe, and pytest prints `11 passed`.

#### What changed

- <a id="step-01.5-justfile-424f0b"></a>`justfile`: the recipe `test`. 4 lines added. [The file](https://github.com/rahuldave/cheesecracker/blob/e86e0c3d10bbe9409674f1c65ce4afdc38e7f76f/justfile).
  - look at `uv run pytest "$@"`

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

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/e86e0c3d10bbe9409674f1c65ce4afdc38e7f76f). [The code after it](https://github.com/rahuldave/cheesecracker/tree/e86e0c3d10bbe9409674f1c65ce4afdc38e7f76f).

<a id="step-02"></a>

## 2 The API server

```bash
just setup
```

The Cheese App talks to its server over HTTP. FastAPI turns Python functions into the answers of HTTP requests, and
uvicorn is the server that runs a FastAPI app. We write an app that answers two requests, and run it on this machine.

[The code before this part](https://github.com/rahuldave/cheesecracker/tree/e86e0c3d10bbe9409674f1c65ce4afdc38e7f76f). [The code after it](https://github.com/rahuldave/cheesecracker/tree/69c3bf65c78cf8e96bf8f9ea1dde29279db1351a).

<a id="step-02.1"></a>

### 2.1 FastAPI, and uvicorn to serve it

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

A patch to [`src/api-service/api/service.py`](#step-02.2-src-api-service-api-service.py-1e1e71).

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

A patch to [`justfile`](#step-02.3-justfile-424f0b).

A patch to [`.gitignore`](#step-02.3-.gitignore-a5cc29).

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

#### What changed

- <a id="step-02.3-.gitignore-a5cc29"></a>`.gitignore`: the log of the server and the number of its process. 4 lines added. [The file](https://github.com/rahuldave/cheesecracker/blob/69c3bf65c78cf8e96bf8f9ea1dde29279db1351a/.gitignore).
  - look at `.serve.pid`
- <a id="step-02.3-justfile-424f0b"></a>`justfile`: the recipes `serve` and `stop`. 26 lines added. [The file](https://github.com/rahuldave/cheesecracker/blob/69c3bf65c78cf8e96bf8f9ea1dde29279db1351a/justfile).
  - look at `uv run uvicorn api.service:app --app-dir src/api-service --host 0.0.0.0 --port 9000 --reload > .serve.log 2>&1 &`

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

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/69c3bf65c78cf8e96bf8f9ea1dde29279db1351a). [The code after it](https://github.com/rahuldave/cheesecracker/tree/69c3bf65c78cf8e96bf8f9ea1dde29279db1351a).

<a id="step-03"></a>

## 3 The distance endpoint, and integration tests

```bash
just setup
```

The app needs an endpoint that does some work, and tests that call it. The endpoint computes the distance of a point
from the origin with `power()`. An integration test sends requests to the app and checks the answers, with no server
and no network.

[The code before this part](https://github.com/rahuldave/cheesecracker/tree/69c3bf65c78cf8e96bf8f9ea1dde29279db1351a). [The code after it](https://github.com/rahuldave/cheesecracker/tree/5862780b44d2f4ec87088c971dec57f23c11139e).

<a id="step-03.1"></a>

### 3.1 /euclidean_distance/, the distance of a point from the origin

The Cheese App asks the server for the distance of a point `(x, y)` from the origin `(0, 0)`. That distance is the
square root of x squared plus y squared.

A patch to [`src/api-service/api/service.py`](#step-03.1-src-api-service-api-service.py-1e1e71).

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

FastAPI's `TestClient` sends requests to the app in the same process, with no server. It needs the package httpx2.
One test of the app is a coroutine, an `async` function, and pytest-asyncio runs it.

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

A patch to [`tests/integration/test_api.py`](#step-03.3-tests-integration-test_api.py-596674).

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

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/5862780b44d2f4ec87088c971dec57f23c11139e). [The code after it](https://github.com/rahuldave/cheesecracker/tree/5862780b44d2f4ec87088c971dec57f23c11139e).

<a id="step-04"></a>

## 4 Coverage

```bash
just setup
```

Twenty-one tests pass, and we still do not know which lines of the app they run. Coverage counts the lines of the
package `api` that run during the tests, and lists the lines that no test ran.

[The code before this part](https://github.com/rahuldave/cheesecracker/tree/5862780b44d2f4ec87088c971dec57f23c11139e). [The code after it](https://github.com/rahuldave/cheesecracker/tree/bfb9c783c67c31254ce8136784738c0f2fc17fb5).

<a id="step-04.1"></a>

### 4.1 pytest-cov, to measure what the tests run

pytest-cov adds the option `--cov` to pytest. It is a tool of the tests, so it goes into the group `test`.

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

A patch to [`justfile`](#step-04.2-justfile-424f0b).

The recipe `cov`, at the end of the `justfile`, runs pytest with `--cov=api --cov-report=term-missing`, and passes
on to pytest any words that you type after `just cov`.

```bash
just cov -q
```

The total is 95%. `service.py` misses line 31, the answer of `/health`, because no test asks for it.

#### What changed

- <a id="step-04.2-justfile-424f0b"></a>`justfile`: the recipe `cov`. 4 lines added. [The file](https://github.com/rahuldave/cheesecracker/blob/bfb9c783c67c31254ce8136784738c0f2fc17fb5/justfile).
  - look at `uv run pytest --cov=api --cov-report=term-missing "$@"`

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

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/bfb9c783c67c31254ce8136784738c0f2fc17fb5). [The code after it](https://github.com/rahuldave/cheesecracker/tree/bfb9c783c67c31254ce8136784738c0f2fc17fb5).

<a id="step-05"></a>

## 5 The formatter and the linter

```bash
just setup
```

Code that works can still be hard to read, and a reader can miss a bug in it. A formatter writes the code in one
layout. A linter checks the code against rules, such as "no import that nothing uses". ruff is both, and it has the
rules of flake8.

[The code before this part](https://github.com/rahuldave/cheesecracker/tree/bfb9c783c67c31254ce8136784738c0f2fc17fb5). [The code after it](https://github.com/rahuldave/cheesecracker/tree/e89e0cfa991257757c887f687087a9c77c093e20).

<a id="step-05.1"></a>

### 5.1 a change in a hurry: the code still works, and reads worse

A change made in a hurry leaves code that works and reads badly. The change adds an import that nothing uses, and takes
out spaces and blank lines. It also makes one line 187 characters long, and adds a function that nothing calls.

A patch to [`src/api-service/api/service.py`](#step-05.1-src-api-service-api-service.py-1e1e71).

A patch to [`src/api-service/api/utils.py`](#step-05.1-src-api-service-api-utils.py-18f1f8).

`service.py` now imports `os`, which nothing uses. `/health` follows `/` with no blank line. The line
`z=power(x,2)+power(y,2)` has no spaces, and the `return` at the end is 187 characters long. `utils.py` has a missing
space, three blank lines, and `unused_function()`.

```bash
just test -q
```

pytest still prints `21 passed`. The tests check what the code does, not how it reads.

#### What changed

- <a id="step-05.1-src-api-service-api-service.py-1e1e71"></a>`src/api-service/api/service.py`: an import that nothing uses, missing spaces and blank lines, and a long line. 3 lines added, 5 removed. [The file](https://github.com/rahuldave/cheesecracker/blob/f77650600c1176caa9aaee86e6cddfccc43fc69c/src/api-service/api/service.py).
  - look at `z=power(x,2)+power(y,2)`
- <a id="step-05.1-src-api-service-api-utils.py-18f1f8"></a>`src/api-service/api/utils.py`: a missing space, extra blank lines, and a function that nothing calls. 6 lines added, 1 removed. [The file](https://github.com/rahuldave/cheesecracker/blob/f77650600c1176caa9aaee86e6cddfccc43fc69c/src/api-service/api/utils.py).
  - look at `def unused_function():`

<details><summary>The diff of <code>src/api-service/api/service.py</code></summary>

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

<details><summary>The diff of <code>src/api-service/api/utils.py</code></summary>

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

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/f77650600c1176caa9aaee86e6cddfccc43fc69c). [The code after it](https://github.com/rahuldave/cheesecracker/tree/f77650600c1176caa9aaee86e6cddfccc43fc69c).

<a id="step-05.2"></a>

### 5.2 ruff, the formatter and the linter, in the group lint

ruff checks the code, and the server does not need it to run. So ruff goes into a group of its own, `lint`.

This change came from running `uv add --group lint ruff`.

```bash
uv run ruff --version
```

uv prints the version of ruff, `ruff 0.17.1` on the day of the class.

#### What changed

- <a id="step-05.2-pyproject.toml-5d07e7"></a>`pyproject.toml`: ruff, in the group `lint`. 3 lines added. It comes from `uv add --group lint ruff`. [The file](https://github.com/rahuldave/cheesecracker/blob/9662828462ba2a21ca04a79c412e83ac070767f6/pyproject.toml).
  - look at `"ruff>=0.17.1",`
- <a id="step-05.2-uv.lock-8356d8"></a>`uv.lock`: the exact version of ruff. 29 lines added. It comes from `uv add --group lint ruff`. [The file](https://github.com/rahuldave/cheesecracker/blob/9662828462ba2a21ca04a79c412e83ac070767f6/uv.lock).

<details><summary>The diff of <code>pyproject.toml</code></summary>

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

A patch to [`pyproject.toml`](#step-05.3-pyproject.toml-5d07e7).

`[tool.ruff]` sets `line-length = 120`, for the formatter and the linter. `[tool.ruff.lint]` selects the rules `E`, `W`
and `F`. They are the errors and warnings of pycodestyle, and the rules of pyflakes, which flake8 checks.

```bash
uv run ruff check
```

ruff finds 2 errors, and fails. `F401` is the import of `os`, which nothing uses. `E501` is the line of 187
characters. ruff says that it can fix the first one.

```bash
uv run ruff format --check
```

ruff says `2 files would be reformatted`, and fails. `--check` changes no file.

#### What changed

- <a id="step-05.3-pyproject.toml-5d07e7"></a>`pyproject.toml`: the settings of ruff, the length of a line and the rules of the linter. 8 lines added. [The file](https://github.com/rahuldave/cheesecracker/blob/7c1670fd52a0f62f414f31d415d10f2efb5a8405/pyproject.toml).
  - look at `select = ["E", "W", "F"]`

<details><summary>The diff of <code>pyproject.toml</code></summary>

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

A patch to [`justfile`](#step-05.4-justfile-424f0b).

The recipe `fmt` runs `uv run ruff format`, and `lint` runs `uv run ruff check`. So `just fmt --check` checks the
layout, and `just lint --fix` fixes what ruff can fix.

```bash
just fmt --check
```

The recipe fails as `ruff format --check` did, with `2 files would be reformatted`.

#### What changed

- <a id="step-05.4-justfile-424f0b"></a>`justfile`: the recipes `fmt` and `lint`. 8 lines added. [The file](https://github.com/rahuldave/cheesecracker/blob/b947593e84b71486902b3ff1c7f9e1458b70e3f7/justfile).
  - look at `uv run ruff check "$@"`

<details><summary>The diff of <code>justfile</code></summary>

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

<a id="step-05.5"></a>

### 5.5 the formatter fixes the spaces, the blank lines and the line breaks

The formatter writes every file in one layout, so nobody fixes spaces by hand.

This change came from running `just fmt`.

ruff prints `2 files reformatted`. In `service.py`, `z=power(x,2)+power(y,2)` now has its spaces, and two blank lines
separate the routes. The long `return` now spreads its dictionary over six lines, one key on each line. In `utils.py`, the blank lines are two.

```bash
just fmt --check
```

ruff prints `6 files already formatted`.

#### What changed

- <a id="step-05.5-src-api-service-api-service.py-1e1e71"></a>`src/api-service/api/service.py`: the spaces, the blank lines, and the long `return` on six lines. 10 lines added, 2 removed. It comes from `just fmt`. [The file](https://github.com/rahuldave/cheesecracker/blob/42642115394d8e19ead4c27bafaf4d90d991b0c9/src/api-service/api/service.py).
  - look at `z = power(x, 2) + power(y, 2)`
- <a id="step-05.5-src-api-service-api-utils.py-18f1f8"></a>`src/api-service/api/utils.py`: the space after the comma, and two blank lines before `unused_function()`. 1 line added, 2 removed. It comes from `just fmt`. [The file](https://github.com/rahuldave/cheesecracker/blob/42642115394d8e19ead4c27bafaf4d90d991b0c9/src/api-service/api/utils.py).
  - look at `def power(base, exponent):`

<details><summary>The diff of <code>src/api-service/api/service.py</code></summary>

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

<details><summary>The diff of <code>src/api-service/api/utils.py</code></summary>

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

This change came from running `just lint --fix || true`.

ruff removes the line `import os` from `service.py`, and still reports the long line.

```bash
just lint
```

ruff finds 1 error, `E501`, and fails. The line of the message is 147 characters long, and no tool can shorten words.

#### What changed

- <a id="step-05.6-src-api-service-api-service.py-1e1e71"></a>`src/api-service/api/service.py`: the import of `os`, removed. 0 lines added, 1 removed. It comes from `just lint --fix || true`. [The file](https://github.com/rahuldave/cheesecracker/blob/cfd2ebb050dda3ce951f0f28e84b9ae09fbe14d5/src/api-service/api/service.py).

<details><summary>The diff of <code>src/api-service/api/service.py</code></summary>

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

A patch to [`src/api-service/api/service.py`](#step-05.7-src-api-service-api-service.py-1e1e71).

The value of `"message"` in the `return` of `calculate_euclidean_distance()` is now shorter than 120 characters.

```bash
just lint
```

ruff prints `All checks passed!`. Note that ruff never reports `unused_function()`. No rule of `E`, `W` or `F` looks for
a function that nothing calls.

#### What changed

- <a id="step-05.7-src-api-service-api-service.py-1e1e71"></a>`src/api-service/api/service.py`: a shorter message. 1 line added, 1 removed. [The file](https://github.com/rahuldave/cheesecracker/blob/693a713a458db01debdd7ebc3b084ed1b0350b08/src/api-service/api/service.py).
  - look at `"message": "This is a very long line that exceeds 120 characters blah",`

<details><summary>The diff of <code>src/api-service/api/service.py</code></summary>

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

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/693a713a458db01debdd7ebc3b084ed1b0350b08). [The code after it](https://github.com/rahuldave/cheesecracker/tree/693a713a458db01debdd7ebc3b084ed1b0350b08).

<a id="step-05.8"></a>

### 5.8 just check runs every check

The layout, the rules and the tests each have a recipe now. The recipe `check` runs all three, and stops at the
first that fails.

A patch to [`justfile`](#step-05.8-justfile-424f0b).

The recipe `check`, at the end of the `justfile`, runs `just fmt --check`, `just lint` and `just test -q`, in that
order.

```bash
just check
```

just prints each command, and the last line is `21 passed`.

#### What changed

- <a id="step-05.8-justfile-424f0b"></a>`justfile`: the recipe `check`. 6 lines added. [The file](https://github.com/rahuldave/cheesecracker/blob/e89e0cfa991257757c887f687087a9c77c093e20/justfile).
  - look at `just fmt --check`

<details><summary>The diff of <code>justfile</code></summary>

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

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/e89e0cfa991257757c887f687087a9c77c093e20). [The code after it](https://github.com/rahuldave/cheesecracker/tree/e89e0cfa991257757c887f687087a9c77c093e20).

<a id="step-06"></a>

## 6 Checks before every commit

```bash
just setup
```

`just check` works only when someone runs it. pre-commit runs checks before every `git commit`, and stops the commit
when a check fails. The checks are in `.pre-commit-config.yaml`.

[The code before this part](https://github.com/rahuldave/cheesecracker/tree/e89e0cfa991257757c887f687087a9c77c093e20). [The code after it](https://github.com/rahuldave/cheesecracker/tree/57ec2e24ecd4946823edbfad27bc2296f80e3847).

<a id="step-06.1"></a>

### 6.1 pre-commit, in the group lint

pre-commit checks the code, and the server does not need it to run. So it goes into the group `lint`, with ruff.

This change came from running `uv add --group lint pre-commit`.

```bash
uv run pre-commit --version
```

uv prints the version of pre-commit.

#### What changed

- <a id="step-06.1-pyproject.toml-5d07e7"></a>`pyproject.toml`: pre-commit, in the group `lint`. 1 line added. It comes from `uv add --group lint pre-commit`. [The file](https://github.com/rahuldave/cheesecracker/blob/f3ad230bf911e0c446eb6752ec2718b910d9e851/pyproject.toml).
  - look at `"pre-commit>=4.7.0",`
- <a id="step-06.1-uv.lock-8356d8"></a>`uv.lock`: the exact versions of pre-commit, and of the packages that it needs. 149 lines added, 1 removed. It comes from `uv add --group lint pre-commit`. [The file](https://github.com/rahuldave/cheesecracker/blob/f3ad230bf911e0c446eb6752ec2718b910d9e851/uv.lock).

<details><summary>The diff of <code>pyproject.toml</code></summary>

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

A patch to [`.pre-commit-config.yaml`](#step-06.2-.pre-commit-config.yaml-e7d14d).

The new file `.pre-commit-config.yaml` names pre-commit-hooks at `v6.0.0`, with seven checks of any file, such as
`trailing-whitespace` and `detect-private-key`. Then it names ruff-pre-commit at `v0.17.1`, with `ruff-check` and
`ruff-format`.

pre-commit checks the files that git tracks, as they are in your folder. It skips a file that git does not track yet.
The first run downloads the checks, which takes a minute.

```bash
uv run pre-commit run --all-files
```

pre-commit prints a line for each check, `Passed` or `Skipped`.

Every file of the project is in a commit here. `check json` says `Skipped`, because the project has no JSON file.
Every other check says `Passed`.

#### What changed

- <a id="step-06.2-.pre-commit-config.yaml-e7d14d"></a>`.pre-commit-config.yaml`: the checks that run before every commit. New, 22 lines. [The file](https://github.com/rahuldave/cheesecracker/blob/009d77653eadfde09e438656cde77b5eac489115/.pre-commit-config.yaml).
  - look at `- id: ruff-check`

<details><summary>The diff of <code>.pre-commit-config.yaml</code></summary>

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

A patch to [`justfile`](#step-06.3-justfile-424f0b).

The recipe `hook`, after the recipe `setup`, runs `uv run pre-commit install`.

```bash
just hook
```

pre-commit prints `pre-commit installed at .git/hooks/pre-commit`. From now on, `git commit` runs the checks first.

#### What changed

- <a id="step-06.3-justfile-424f0b"></a>`justfile`: the recipe `hook`. 4 lines added. [The file](https://github.com/rahuldave/cheesecracker/blob/57ec2e24ecd4946823edbfad27bc2296f80e3847/justfile).
  - look at `uv run pre-commit install`

<details><summary>The diff of <code>justfile</code></summary>

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

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/57ec2e24ecd4946823edbfad27bc2296f80e3847). [The code after it](https://github.com/rahuldave/cheesecracker/tree/57ec2e24ecd4946823edbfad27bc2296f80e3847).

<a id="step-07"></a>

## 7 The server in Docker

```bash
just setup
```

The server runs on this machine, with its `.venv` and this version of macOS or Linux. Production is another machine,
such as a server on Google Cloud. A Docker image holds the server with everything it needs, so it runs the same on
any machine with Docker. The run image is the image that production runs, our deployment.

[The code before this part](https://github.com/rahuldave/cheesecracker/tree/57ec2e24ecd4946823edbfad27bc2296f80e3847). [The code after it](https://github.com/rahuldave/cheesecracker/tree/d444e0387d256c62024581e98f59d670cbc93296).

<a id="step-07.1"></a>

### 7.1 what docker build leaves out

`docker build` sends the files of the folder to Docker. The folder has `.venv`, the caches of the tools, git and maybe
secrets. None of them belongs in an image. `.dockerignore` lists them.

A patch to [`.dockerignore`](#step-07.1-.dockerignore-05a721).

The new file `.dockerignore` lists `.venv/`, the caches of pytest, ruff and coverage, `.git/`, `secrets/` and `.env`.

```bash
cat .dockerignore
```

`cat` prints the list.

#### What changed

- <a id="step-07.1-.dockerignore-05a721"></a>`.dockerignore`: what `docker build` never sends to Docker. New, 10 lines. [The file](https://github.com/rahuldave/cheesecracker/blob/2a28a5c704d6d9ed8200aeb8962aa01e5c2ac552/.dockerignore).
  - look at `.venv/`

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

A patch to [`Dockerfile`](#step-07.2-Dockerfile-6651dd).

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

A patch to [`justfile`](#step-07.3-justfile-424f0b).

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

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/d444e0387d256c62024581e98f59d670cbc93296). [The code after it](https://github.com/rahuldave/cheesecracker/tree/d444e0387d256c62024581e98f59d670cbc93296).

<a id="step-08"></a>

## 8 System tests

```bash
just setup
```

The integration tests call the app in the same process as the tests. They cannot tell whether the image starts, or
whether the server answers on its port. A system test sends real HTTP requests to the running server, as the Cheese
App does. Here the server is the run image, our deployment, which `just docker-serve` starts.

[The code before this part](https://github.com/rahuldave/cheesecracker/tree/d444e0387d256c62024581e98f59d670cbc93296). [The code after it](https://github.com/rahuldave/cheesecracker/tree/277d54eebce4a1727ed3d8a5c46570ca4a95f75d).

<a id="step-08.1"></a>

### 8.1 requests, for tests that send real HTTP requests

The system tests send their requests with the package requests. It is a tool of the tests, so it goes into the group
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

A patch to [`pyproject.toml`](#step-08.2-pyproject.toml-5d07e7).

`testpaths` in `[tool.pytest.ini_options]` now names `tests/unit` and `tests/integration`, in place of `tests`. A
folder that pytest gets on its command line, such as `tests/system`, still runs.

```bash
just test -q
```

pytest prints `21 passed`, as before.

#### What changed

- <a id="step-08.2-pyproject.toml-5d07e7"></a>`pyproject.toml`: the two folders of tests that run by default. 1 line added, 1 removed. [The file](https://github.com/rahuldave/cheesecracker/blob/c8302b4cff8bd9fc480ba4f74a0c7d34c769fef6/pyproject.toml).
  - look at `testpaths = ["tests/unit", "tests/integration"]`

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

A patch to [`tests/system/test_system_api.py`](#step-08.3-tests-system-test_system_api.py-780a11).

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

A patch to [`justfile`](#step-08.4-justfile-424f0b).

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

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/277d54eebce4a1727ed3d8a5c46570ca4a95f75d). [The code after it](https://github.com/rahuldave/cheesecracker/tree/277d54eebce4a1727ed3d8a5c46570ca4a95f75d).

<a id="step-09"></a>

## 9 The dev image

```bash
just setup
```

The checks run on this machine. Another person, or a CI service, runs them on another system, with other versions of
the tools. The dev image holds the whole project with every group of dependencies, and uv, just and git. So the
checks run in the same Linux, anywhere Docker runs.

[The code before this part](https://github.com/rahuldave/cheesecracker/tree/277d54eebce4a1727ed3d8a5c46570ca4a95f75d). [The code after it](https://github.com/rahuldave/cheesecracker/tree/0701e9244ab2384538eda4d623e1d047ff3f8258).

<a id="step-09.1"></a>

### 9.1 the dev image, to work on the project in Linux

The dev image is a fourth stage of the `Dockerfile`, `dev`. It takes uv from the stage `uv`, and nothing from the
run image.

A patch to [`Dockerfile`](#step-09.1-Dockerfile-6651dd).

The stage `dev` installs git and curl, copies uv, and installs just as a package of PyPI. It installs every group of
`uv.lock` into `/opt/venv`, copies the project into `/work`, and starts a shell. `/opt/venv` is outside `/work`, so a
`.venv` of this machine in a mounted folder is never used.

```bash
docker build --target dev --tag cheesecracker-dev .
```

Near the end, the line `naming to docker.io/library/cheesecracker-dev` names the image.

```bash
docker image ls cheesecracker-dev
```

The image is about 353 MB, twice the run image, because it has every tool.

#### What changed

- <a id="step-09.1-Dockerfile-6651dd"></a>`Dockerfile`: the stage `dev`, and its lines in the comment at the top. 26 lines added. [The file](https://github.com/rahuldave/cheesecracker/blob/ccf62d330f28c6eec4c0b993e1a71e2c11a8b5af/Dockerfile).
  - look at `FROM python:3.12-slim-bookworm AS dev`

<details><summary>The diff of <code>Dockerfile</code></summary>

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

<a id="step-09.2"></a>

### 9.2 just docker-shell and just docker-test

Two recipes run the dev image with this folder mounted at `/work`. An edit here shows in there at once, so the image
needs no rebuild after a change of the code. `docker-shell` opens a shell in the image. `docker-test` runs the recipe
that you name in the image, or `just check` if you name none.

A patch to [`justfile`](#step-09.2-justfile-424f0b).

`docker-shell` runs `docker run --rm -it -v "$PWD":/work cheesecracker-dev`. `-v` mounts this folder at `/work`, and
`-it` gives the shell a terminal. `docker-test` runs `just` in the image, with its words, or with `check`.

```bash
just docker-test
```

`just check` runs in Linux. The last line is `21 passed`. The system tests do not run, because `just check` leaves them
out.

```bash
just docker-test test tests/unit -q
```

The eleven unit tests pass in Linux.

#### What changed

- <a id="step-09.2-justfile-424f0b"></a>`justfile`: the recipes `docker-shell` and `docker-test`. 8 lines added. [The file](https://github.com/rahuldave/cheesecracker/blob/0701e9244ab2384538eda4d623e1d047ff3f8258/justfile).
  - look at `docker run --rm -v "$PWD":/work cheesecracker-dev just "${@:-check}"`

<details><summary>The diff of <code>justfile</code></summary>

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

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/0701e9244ab2384538eda4d623e1d047ff3f8258). [The code after it](https://github.com/rahuldave/cheesecracker/tree/0701e9244ab2384538eda4d623e1d047ff3f8258).

<a id="step-10"></a>

## 10 A new endpoint

```bash
just setup
```

The Cheese App needs to add two numbers. The endpoint `/add/` takes `x` and `y`, and returns their sum. An integration
test calls it. Then every check of the project runs again, on this machine, in the dev image, and against a new run
image.

A patch to [`src/api-service/api/service.py`](#step-10-src-api-service-api-service.py-1e1e71).

A patch to [`tests/integration/test_api.py`](#step-10-tests-integration-test_api.py-596674).

`add_numbers()`, at the end of `service.py`, answers `/add/` with `{"result": x + y}`. `test_add_endpoint()`, at the
end of `test_api.py`, asks for `/add/?x=10&y=20`, and expects 30.

```bash
just check
```

The checks pass on this machine. The integration tests are now eleven.

```bash
just docker-test
```

The same checks pass in the dev image.

The run image still has the old code, because an image is built once and does not change. Build it again, and start
its server.

```bash
just docker-build run
just docker-serve
curl -s "localhost:9000/add/?x=10&y=20"
```

The server in Docker answers `{"result":30.0}`.

```bash
just test-system -q
```

The system tests print `5 passed` against the new run image.

```bash
just docker-stop
```

### What changed

- <a id="step-10-src-api-service-api-service.py-1e1e71"></a>`src/api-service/api/service.py`: the endpoint `/add/`. 6 lines added. [The file](https://github.com/rahuldave/cheesecracker/blob/e706255d9c0fee3b6fded40376e7ac8829d1654a/src/api-service/api/service.py).
  - look at `return {"result": x + y}`
- <a id="step-10-tests-integration-test_api.py-596674"></a>`tests/integration/test_api.py`: the test of `/add/`. 6 lines added. [The file](https://github.com/rahuldave/cheesecracker/blob/e706255d9c0fee3b6fded40376e7ac8829d1654a/tests/integration/test_api.py).
  - look at `response = client.get("/add/?x=10&y=20")`

<details><summary>The diff of <code>src/api-service/api/service.py</code></summary>

```diff
@@ -44,3 +44,9 @@ async def calculate_euclidean_distance(x: float = 1, y: float = 2):
         "distance": result,
         "message": "This is a very long line that exceeds 120 characters blah",
     }
+
+
+@app.get("/add/")
+async def add_numbers(x: float = 0, y: float = 0):
+    """Add two numbers"""
+    return {"result": x + y}
```

</details>

<details><summary>The diff of <code>tests/integration/test_api.py</code></summary>

```diff
@@ -87,3 +87,9 @@ async def test_api_health_check():
     assert response.status_code == 200
     # API should respond quickly
     assert response.elapsed.total_seconds() < 1 if hasattr(response, "elapsed") else True
+
+
+def test_add_endpoint():
+    response = client.get("/add/?x=10&y=20")
+    assert response.status_code == 200
+    assert response.json()["result"] == 30
```

</details>

[This change on GitHub](https://github.com/rahuldave/cheesecracker/compare/0701e9244ab2384538eda4d623e1d047ff3f8258...e706255d9c0fee3b6fded40376e7ac8829d1654a). [The code after it](https://github.com/rahuldave/cheesecracker/tree/e706255d9c0fee3b6fded40376e7ac8829d1654a).
