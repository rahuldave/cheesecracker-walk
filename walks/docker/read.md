<!-- Made by the walk skill's render.py from the facts of the walk. Do not edit it by hand: run just walk-update again. -->

# cheesecracker in Docker

**Read the tutorial.** Each change explained, with the code to look at. [Build it yourself](build.md) is the other way to read it.

Each part ends with a section called **What changed**. It lists every file that the part changes, with a link to the file, and then the whole diff of each file, folded. A lockfile and a binary file have a link alone. The text above it explains the change.

We put cheesecracker, the API server of the Cheese App, into Docker.

[Docker](https://docs.docker.com/) runs programs in containers. A container is a process with its own files, cut off
from the rest of the machine. It starts from an image, a small Linux system with the program and everything that the
program needs. A file named `Dockerfile` lists the instructions that build an image. `docker build` makes the image,
and `docker run` starts a container from it. On macOS and Windows, the app Docker Desktop runs Docker.

An image runs the same on any machine with Docker, so the server runs for the real users of the app as it runs here.
We build two images from one lockfile, `uv.lock`, which names the exact version of each package of the project. So
both images hold the same versions. The run image is the server as it runs for the
real users of the app, which we call production. It is our deployment.

The dev image holds the project with every tool. This tutorial is the part of the whole tutorial of cheesecracker that adds Docker.

The images depend on the groups of dependencies of the project. The group `test` holds the tools of the tests, and the
group `lint` holds ruff and pre-commit. The run image installs no group, and the dev image installs every group. The
tutorial uv, beside this one, shows how the groups came to be.

Two tools run the project on this machine. [uv](https://docs.astral.sh/uv/) installs Python and the packages of the
project into the folder `.venv`, at the exact versions of `uv.lock`. [just](https://just.systems/) runs each task of
the project by a short name. Each task is a recipe in the file `justfile`. `just setup` runs `uv sync --locked`,
which makes uv install everything into `.venv`, and `just` alone lists the recipes.

The code is in the repository [rahuldave/cheesecracker](https://github.com/rahuldave/cheesecracker) on GitHub. When the
project is done, the server's code is in `src/api-service/api/`, and its tests are in `tests/unit/`, `tests/integration/` and
`tests/system/`. `pyproject.toml` and `uv.lock` list the Python packages that the project needs. `justfile` holds its
recipes, short names for the commands of the project. `Dockerfile` holds the instructions that build its two Docker images,
the run image of the server and the dev image with every tool.

The repository keeps every saved version of the project, in the order of the whole tutorial of cheesecracker. That tutorial has many
parts, and this tutorial follows some of them.

On the website, each page covers one part of the tutorial. The top of the page links to the code as it is when the
part starts. The bottom links to the code as it is when the part ends.

We make the project in moves. A move is one small change, made by one patch or by one command. You make each move,
and the page checks your files against it.

The tutorial starts from the project as the whole tutorial leaves it before Docker. The server answers `/`, `/health`
and `/euclidean_distance/`. It has unit and integration tests, coverage, ruff, and pre-commit. Each task is a recipe,
and `just check` runs the format, the lint and the tests.

<a id="step-07.1"></a>

## 7.1 What docker build leaves out

This walk goes through [rahuldave/cheesecracker](https://github.com/rahuldave/cheesecracker) on GitHub. This part starts from the code at `step-06`. [The code before this part](https://github.com/rahuldave/cheesecracker/tree/step-06)

```bash
just setup
```

Our first file for Docker, `.dockerignore`, lists the files that never go into an image.

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

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/2a28a5c704d6d9ed8200aeb8962aa01e5c2ac552).

This part ends at the code at `2a28a5c704d6`, partway through `step-07`. [The code after this part](https://github.com/rahuldave/cheesecracker/tree/2a28a5c704d6d9ed8200aeb8962aa01e5c2ac552)

<a id="step-07.2-to-step-07.3"></a>

## 7.2 to 7.3 The run image

This walk goes through [rahuldave/cheesecracker](https://github.com/rahuldave/cheesecracker) on GitHub. This part starts from the code at `2a28a5c704d6`, partway through `step-07`. [The code before this part](https://github.com/rahuldave/cheesecracker/tree/2a28a5c704d6d9ed8200aeb8962aa01e5c2ac552)

```bash
just setup
```

The run image holds Python, the dependencies of the server and its code, and starts the server. Three recipes build
it, start it and stop it.

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

A system test sends real HTTP requests to a running server, as a user of the app does. The run image is our
deployment, so the system tests send their requests to the server in it.

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

<a id="step-09"></a>

## 9 The dev image

This walk goes through [rahuldave/cheesecracker](https://github.com/rahuldave/cheesecracker) on GitHub. This part starts from the code at `step-08`. [The code before this part](https://github.com/rahuldave/cheesecracker/tree/step-08)

The dev image runs the checks in Linux, anywhere Docker runs.

```bash
just setup
```

The checks run on this machine. Another person, or a CI service, runs them on another system, with other versions of
the tools. The dev image holds the whole project with every group of dependencies, and uv, just and git. So the
checks run in the same Linux, anywhere Docker runs.

<a id="step-09.1"></a>

### 9.1 the dev image, to work on the project in Linux

The dev image is a fourth stage of the `Dockerfile`, `dev`. It takes uv from the stage `uv`, and nothing from the
run image.

[± See the change to `Dockerfile`](#step-09.1-Dockerfile-6651dd-diff)

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

<a id="step-09.1-Dockerfile-6651dd-diff"></a>

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

[± See the change to `justfile`](#step-09.2-justfile-424f0b-diff)

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

<a id="step-09.2-justfile-424f0b-diff"></a>

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

[This change on GitHub](https://github.com/rahuldave/cheesecracker/commit/0701e9244ab2384538eda4d623e1d047ff3f8258).

This part ends at the code at `step-09`. [The code after this part](https://github.com/rahuldave/cheesecracker/tree/step-09)

<a id="step-10"></a>

## 10 A change, through both images

This walk goes through [rahuldave/cheesecracker](https://github.com/rahuldave/cheesecracker) on GitHub. This part starts from the code at `step-09`. [The code before this part](https://github.com/rahuldave/cheesecracker/tree/step-09)

A change of the code goes through both images. The dev image checks it, and the run image has to be built again.

```bash
just setup
```

The Cheese App needs to add two numbers. The endpoint `/add/` takes `x` and `y`, and returns their sum. An integration
test calls it. Then every check of the project runs again, on this machine, in the dev image, and against a new run
image.

[± See the change to `src/api-service/api/service.py`](#step-10-src-api-service-api-service.py-1e1e71-diff)

[± See the change to `tests/integration/test_api.py`](#step-10-tests-integration-test_api.py-596674-diff)

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

<a id="step-10-src-api-service-api-service.py-1e1e71-diff"></a>

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

<a id="step-10-tests-integration-test_api.py-596674-diff"></a>

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

[This change on GitHub](https://github.com/rahuldave/cheesecracker/compare/0701e9244ab2384538eda4d623e1d047ff3f8258...e706255d9c0fee3b6fded40376e7ac8829d1654a).

This part ends at the code at `step-10`. [The code after this part](https://github.com/rahuldave/cheesecracker/tree/step-10)
