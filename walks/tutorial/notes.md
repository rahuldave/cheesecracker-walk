# The walk through cheesecracker, step by step

This file holds the notes of every step of a walk. timewalk shows the notes of the current step in the column on the
right of the page.

- **A command** is a line that starts with `$ `. A click on it types the command into the terminal of the
  step, and you press Enter to run it. With **run on click** on, a click also runs it.
- **Other tabs:** a line that starts with `runs$ ` goes to the **Runs** tab, for a command that takes a
  while. A line that starts with `main$ ` goes to the **Main** tab, in `repo/`.
- **A cue** is a line that starts with `> `. The notes column shows it shaded.
- **Edit** at the top of the column changes the notes of the step, and **Save** writes them into
  this file. Commit it to your fork to keep your notes.
- **`$ just setup` starts every step.** It is the project's own recipe, at the step. It makes the environment
  of the step, and what the step needs.
- **A move keeps your work.** Before a move, timewalk keeps your edits and commits in `worktree/` on a branch
  `timewalk/saved/<step>`, and the page says how to get them back. In a `worktree/` that an older timewalk made, a
  move with edits asks first, and stashes them.

## step-00 Where we start

$ just setup

The project is empty. It has a `pyproject.toml` with no dependencies, an empty `uv.lock`, a `README.md`, a
`.gitignore`, and a `justfile` with one recipe, `setup`. `just setup` runs `uv sync --locked`. uv is the tool that installs Python and
packages. The command makes the folder `.venv`, with Python 3.12 and every package of `uv.lock` in it. A second run changes nothing.

Two settings of `pyproject.toml`, in the table `[tool.uv]`, matter for the whole project. `package = false` says that
cheesecracker is an application. uv installs its dependencies, and does not install the project as a package.
`default-groups = "all"` makes `uv sync` and `uv run` install every group of dependencies, such as the tools of the
tests.

$ just --list

`just --list` prints the one recipe, `setup`, with its comment.

files:
- file `pyproject.toml`: the name of the project, no dependencies yet, and the table `[tool.uv]`.
  show: `default-groups = "all"`
- file `justfile`: the recipe `setup`, and `default`, which lists the recipes.
  show: `uv sync --locked`
- file `.python-version`: the version of Python that uv installs, 3.12.
- file `.gitignore`: the environment, the caches of the tools, and the secrets, which git never takes.
- file `README.md`: what the project is, and how to set it up.
- file `uv.lock`: the lockfile, with the project and no packages yet.

## step-01 power(), and its unit tests

$ just setup

The server will compute distances, and it needs a function that raises a number to a power. We write the function
first, and tests that call it alone. A unit test calls one function, with no server and no network, so it runs in
milliseconds.

### step-01.1 power(), the first function of the API

The code of the API is a Python package, `api`, in the folder `src/api-service/`. Its first function, `power()`,
returns its first argument raised to the power of its second.

patch$ src/api-service/api/__init__.py
patch$ src/api-service/api/utils.py

::: do
Make the folder `src/api-service/api/`, and apply both patches. `__init__.py` has one comment, and makes the folder a
package. In `utils.py`, read the function `power()` after the docstring of the module.
:::

::: watch
The new file `src/api-service/api/utils.py` has a docstring and one function, `power(base, exponent)`. It returns
`base**exponent`. `src/api-service/api/__init__.py` has one comment, and makes `api` a package.
:::

The package is not on the import path of the project yet. So the check runs Python in the folder `src/api-service/`,
where `api` is a folder of its own.

$ uv run --directory src/api-service python -c "from api.utils import power; print(power(2, 3))"

Python prints `8`, which is 2 to the power of 3.

files:
- file `src/api-service/api/utils.py`: the function `power()`.
  show: `return base**exponent`
- file `src/api-service/api/__init__.py`: one comment, which makes the folder a package.

### step-01.2 pytest, in the dependency group test

A test calls `power()` and compares the result with the answer that we expect. pytest finds the tests of a project,
runs them, and reports each failure. The tests need pytest, but the server does not. So pytest goes into a group of
dependencies of its own, `test`.

do$ uv add --group test pytest

The command adds pytest to the group `test` of `pyproject.toml`. It writes the exact versions of pytest and of the
packages that it needs into `uv.lock`.

$ uv run pytest --version

uv prints the version of pytest, `pytest 9.1.1` on the day of the class.

files:
- diff `pyproject.toml`: pytest, in the group `test`.
  show: `"pytest>=9.1.1",`
- diff `uv.lock`: the exact version of pytest, and of the packages that it needs.

### step-01.3 pytest finds the tests, and the package api

pytest looks for tests in every folder of the project, `.venv` too, which is slow. Also, the tests cannot import
`api`, because the folder `src/api-service/` is not on the import path. Two settings of pytest, in `pyproject.toml`, solve both problems.

patch$ pyproject.toml

::: do
Read the patch. It adds the table `[tool.pytest.ini_options]` at the end of `pyproject.toml`.
:::

::: watch
The new table `[tool.pytest.ini_options]` at the end of `pyproject.toml` has two settings. `testpaths` makes pytest
look for tests in `tests/` only. `pythonpath` puts `src/api-service/` on the import path, so a test can import `api`.
:::

$ uv run pytest

pytest collects 0 items, and fails with the code 5, because there are no tests yet. It also warns that it found no
files in `tests/`, which does not exist yet.

files:
- diff `pyproject.toml`: the table `[tool.pytest.ini_options]`, with the folder of the tests and the folder of the package.
  show: `pythonpath = ["src/api-service"]`

### step-01.4 unit tests of power()

We do not know whether `power()` gives the right answer for every kind of input. Eleven tests try it with integers,
zero, one, negative numbers, floats, the special values of floats, and inputs of the wrong type.

patch$ tests/unit/test_utils.py

::: do
Make the folder `tests/unit/`, and apply the patch. Read the class `TestPower`. Each method is one test, and each
`assert` compares a result of `power()` with its answer.
:::

::: watch
The new file `tests/unit/test_utils.py` has the class `TestPower`, with eleven tests. Each `assert` compares a result
of `power()` with its answer. `test_power_invalid_inputs` expects a `TypeError` for a string, `None`, a list or a
dictionary.
:::

$ uv run pytest -q

pytest prints `11 passed`. So `power()` gives the answer that each test expects.

files:
- file `tests/unit/test_utils.py`: eleven unit tests of `power()`.
  show: `assert power(2, 3) == 8`

### step-01.5 just test runs the tests

The command `uv run pytest` is long, and every task of the project gets a recipe in the `justfile`, so that one
short command runs it. The recipe `test` runs pytest through uv, and passes on to pytest any words that you type after `just test`.

patch$ justfile

::: do
Read the recipe `test` at the end of the `justfile`.
:::

::: watch
The recipe `test`, at the end of the `justfile`, runs `uv run pytest "$@"`. `"$@"` stands for the words after
`just test`, so `just test tests/unit` runs the unit tests alone.
:::

$ just test -q

just prints the command of the recipe, and pytest prints `11 passed`.

files:
- diff `justfile`: the recipe `test`.
  show: `uv run pytest "$@"`

## step-02 The API server

$ just setup

The Cheese App talks to its server over HTTP. FastAPI turns Python functions into the answers of HTTP requests, and
uvicorn is the server that runs a FastAPI app. We write an app that answers two requests, and run it on this machine.

### step-02.1 FastAPI, and uvicorn to serve it

The server needs FastAPI and uvicorn to run, so they are dependencies of the project itself, and not of a group.

do$ uv add fastapi uvicorn

The command adds both packages to `dependencies` in `pyproject.toml`, and writes their versions into `uv.lock`.

$ uv run uvicorn --version

uvicorn prints its version, and the version of Python that runs it.

files:
- diff `pyproject.toml`: FastAPI and uvicorn, the dependencies of the server.
  show: `"fastapi>=0.143.2",`
- diff `uv.lock`: the exact versions of FastAPI, uvicorn, and the packages that they need.

### step-02.2 the API server, with / and /health

The app needs a first answer, and a way to tell that it runs. `service.py` makes the FastAPI app, and answers two
requests. The request `/` gets a welcome, and `/health` gets the status of the server.

patch$ src/api-service/api/service.py

::: do
Apply the patch, and read the file from the top. It makes `app`, and adds the CORS middleware, which lets a web page
from any address call the server. Then it defines the two routes after the comment `# Routes`.
:::

::: watch
The new file `src/api-service/api/service.py` makes `app`, with a title and a version. The CORS middleware lets a web
page from any address call the server. After the comment `# Routes`, `get_index()` answers `/`, and `health_check()`
answers `/health`.
:::

$ uv run --directory src/api-service python -c "from api.service import app; print([r.path for r in app.routes])"

Python prints the routes of the app. The last two are `/` and `/health`. FastAPI adds the first four, for its
documentation pages.

files:
- file `src/api-service/api/service.py`: the FastAPI app, its CORS middleware, and the routes `/` and `/health`.
  show: `return {"status": "healthy"}`

### step-02.3 just serve runs the server on this machine

The app does nothing until a server runs it. The recipe `serve` runs uvicorn at port 9000. `--app-dir` tells uvicorn
where the package `api` is, and `--reload` restarts the server when a file of the code changes.

patch$ justfile

::: do
Read the recipe `serve` at the end of the `justfile`.
:::

::: watch
The recipe `serve`, at the end of the `justfile`, runs `uvicorn api.service:app` at port 9000 of every address of this
machine. `api.service:app` names the object `app` in the module `api.service`.
:::

The server runs until you stop it, so it gets a terminal of its own. Leave it running.

runs$ just serve

In another terminal, ask the server for its health.

$ curl -s localhost:9000/health

The server answers `{"status":"healthy"}`. Open `http://localhost:9000/docs` in a browser to see the pages that
FastAPI makes for each route.

files:
- diff `justfile`: the recipe `serve`.
  show: `uv run uvicorn api.service:app --app-dir src/api-service --host 0.0.0.0 --port 9000 --reload`

## step-03 The distance endpoint, and integration tests

$ just setup

The app needs an endpoint that does some work, and tests that call it. The endpoint computes the distance of a point
from the origin with `power()`. An integration test sends requests to the app and checks the answers, with no server
and no network.

### step-03.1 /euclidean_distance/, the distance of a point from the origin

The Cheese App asks the server for the distance of a point `(x, y)` from the origin `(0, 0)`. That distance is the
square root of x squared plus y squared.

patch$ src/api-service/api/service.py

::: do
Read the two imports after `CORSMiddleware`, and the new function at the end of the file.
:::

::: watch
`service.py` now imports `math`, and `power()` from `api.utils`. The new function at the end,
`calculate_euclidean_distance()`, answers `/euclidean_distance/`. It takes `x` and `y`, with 1 and 2 as defaults,
and returns them with the distance.
:::

If `just serve` still runs, it has restarted with the new code. Ask it for the distance of the point (3, 4).

$ curl -s "localhost:9000/euclidean_distance/?x=3&y=4"

The server answers `{"x":3.0,"y":4.0,"distance":5.0}`.

files:
- diff `src/api-service/api/service.py`: the imports of `math` and `power()`, and the endpoint `/euclidean_distance/`.
  show: `z = power(x, 2) + power(y, 2)`

### step-03.2 httpx2 and pytest-asyncio, for tests of the app

FastAPI's `TestClient` sends requests to the app in the same process, with no server. It needs the package httpx2.
One test of the app is a coroutine, an `async` function, and pytest-asyncio runs it.

do$ uv add --group test httpx2 pytest-asyncio

The command adds both packages to the group `test`, and writes their versions into `uv.lock`.

$ just test -q

pytest still prints `11 passed`. The new packages change no test.

files:
- diff `pyproject.toml`: httpx2 and pytest-asyncio, in the group `test`.
  show: `"httpx2>=2.13.1",`
- diff `uv.lock`: the exact versions of httpx2, pytest-asyncio, and the packages that they need.

### step-03.3 integration tests of the API

A unit test of `power()` cannot tell whether the app routes a request to the right function, or reads its
parameters, or answers in JSON. Ten integration tests send requests to the app through `TestClient`, and check each
answer.

patch$ tests/integration/test_api.py

::: do
Make the folder `tests/integration/`, and apply the patch. Read the line `client = TestClient(app)`, then the classes
`TestAPIEndpoints` and `TestCORS`.
:::

::: watch
The new file `tests/integration/test_api.py` makes `client = TestClient(app)`. The class `TestAPIEndpoints` checks
the welcome, the distance of (3, 4) and the defaults. It also checks a route that does not exist (404), and a method
that is not allowed (405). The class `TestCORS` checks the CORS headers.
:::

$ just test -q

pytest prints `21 passed`, the eleven unit tests and the ten integration tests.

files:
- file `tests/integration/test_api.py`: ten integration tests, through FastAPI's `TestClient`.
  show: `response = client.get("/euclidean_distance/?x=3&y=4")`

## step-04 Coverage

$ just setup

Twenty-one tests pass, and we still do not know which lines of the app they run. Coverage counts the lines of the
package `api` that run during the tests, and lists the lines that no test ran.

### step-04.1 pytest-cov, to measure what the tests run

pytest-cov adds the option `--cov` to pytest. It is a tool of the tests, so it goes into the group `test`.

do$ uv add --group test pytest-cov

The check runs the unit tests alone, with coverage of the package `api`.

$ uv run pytest tests/unit --cov=api -q

The table shows `utils.py` at 100% and `service.py` at 0%. The unit tests call `power()`, and never the app.

files:
- diff `pyproject.toml`: pytest-cov, in the group `test`.
  show: `"pytest-cov>=7.1.0",`
- diff `uv.lock`: the exact versions of pytest-cov, and of coverage, which it uses.

### step-04.2 just cov shows which lines the tests run

The recipe `cov` runs every test with coverage, and adds the column `Missing`, with the numbers of the lines that no
test ran.

patch$ justfile

::: do
Read the recipe `cov` at the end of the `justfile`.
:::

::: watch
The recipe `cov`, at the end of the `justfile`, runs pytest with `--cov=api --cov-report=term-missing`, and passes
on to pytest any words that you type after `just cov`.
:::

$ just cov -q

The total is 95%. `service.py` misses line 31, the answer of `/health`, because no test asks for it.

files:
- diff `justfile`: the recipe `cov`.
  show: `uv run pytest --cov=api --cov-report=term-missing "$@"`

## step-05 The formatter and the linter

$ just setup

Code that works can still be hard to read, and a reader can miss a bug in it. A formatter writes the code in one
layout. A linter checks the code against rules, such as "no import that nothing uses". ruff is both, and it has the
rules of flake8.

### step-05.1 a change in a hurry: the code still works, and reads worse

A change made in a hurry leaves code that works and reads badly. The change adds an import that nothing uses, and takes
out spaces and blank lines. It also makes one line 187 characters long, and adds a function that nothing calls.

patch$ src/api-service/api/service.py
patch$ src/api-service/api/utils.py

::: do
Apply both patches. In `service.py`, read the line `import os`, the line `z=power(x,2)+power(y,2)`, and the `return` at
the end. In `utils.py`, read `def power(base,exponent):` and the new function `unused_function()`.
:::

::: watch
`service.py` now imports `os`, which nothing uses. `/health` follows `/` with no blank line. The line
`z=power(x,2)+power(y,2)` has no spaces, and the `return` at the end is 187 characters long. `utils.py` has a missing
space, three blank lines, and `unused_function()`.
:::

$ just test -q

pytest still prints `21 passed`. The tests check what the code does, not how it reads.

files:
- diff `src/api-service/api/service.py`: an import that nothing uses, missing spaces and blank lines, and a long line.
  show: `z=power(x,2)+power(y,2)`
- diff `src/api-service/api/utils.py`: a missing space, extra blank lines, and a function that nothing calls.
  show: `def unused_function():`

### step-05.2 ruff, the formatter and the linter, in the group lint

ruff checks the code, and the server does not need it to run. So ruff goes into a group of its own, `lint`.

do$ uv add --group lint ruff

$ uv run ruff --version

uv prints the version of ruff, `ruff 0.17.1` on the day of the class.

files:
- diff `pyproject.toml`: ruff, in the group `lint`.
  show: `"ruff>=0.17.1",`
- diff `uv.lock`: the exact version of ruff.

### step-05.3 ruff's settings: lines of 120 characters, and flake8's rules

By default, ruff allows lines of up to 88 characters, and this project allows 120. Its default rules also leave out
some rules of flake8. Two tables in `pyproject.toml` set both.

patch$ pyproject.toml

::: do
Read the tables `[tool.ruff]` and `[tool.ruff.lint]` at the end of `pyproject.toml`.
:::

::: watch
`[tool.ruff]` sets `line-length = 120`, for the formatter and the linter. `[tool.ruff.lint]` selects the rules `E`, `W`
and `F`. They are the errors and warnings of pycodestyle, and the rules of pyflakes, which flake8 checks.
:::

$ uv run ruff check

ruff finds 2 errors, and fails. `F401` is the import of `os`, which nothing uses. `E501` is the line of 187
characters. ruff says that it can fix the first one.

$ uv run ruff format --check

ruff says `2 files would be reformatted`, and fails. `--check` changes no file.

files:
- diff `pyproject.toml`: the settings of ruff, the length of a line and the rules of the linter.
  show: `select = ["E", "W", "F"]`

### step-05.4 just fmt and just lint run ruff

Two recipes give the formatter and the linter short names. Each passes on to ruff any words that you type after it.

patch$ justfile

::: do
Read the recipes `fmt` and `lint` at the end of the `justfile`.
:::

::: watch
The recipe `fmt` runs `uv run ruff format`, and `lint` runs `uv run ruff check`. So `just fmt --check` checks the
layout, and `just lint --fix` fixes what ruff can fix.
:::

$ just fmt --check

The recipe fails as `ruff format --check` did, with `2 files would be reformatted`.

files:
- diff `justfile`: the recipes `fmt` and `lint`.
  show: `uv run ruff check "$@"`

### step-05.5 the formatter fixes the spaces, the blank lines and the line breaks

The formatter writes every file in one layout, so nobody fixes spaces by hand.

do$ just fmt

ruff prints `2 files reformatted`. In `service.py`, `z=power(x,2)+power(y,2)` now has its spaces, and two blank lines
separate the routes. The long `return` now spreads its dictionary over six lines, one key on each line. In `utils.py`, the blank lines are two.

$ just fmt --check

ruff prints `6 files already formatted`.

files:
- diff `src/api-service/api/service.py`: the spaces, the blank lines, and the long `return` on six lines.
  show: `z = power(x, 2) + power(y, 2)`
- diff `src/api-service/api/utils.py`: the space after the comma, and two blank lines before `unused_function()`.
  show: `def power(base, exponent):`

### step-05.6 the linter removes the import that nothing uses

ruff can remove an import that nothing uses, because the change cannot alter what the code does. The linter still fails
after the fix, because one error is left. `|| true` makes the command count as a success all the same.

do$ just lint --fix || true

ruff removes the line `import os` from `service.py`, and still reports the long line.

$ just lint

ruff finds 1 error, `E501`, and fails. The line of the message is 147 characters long, and no tool can shorten words.

files:
- diff `src/api-service/api/service.py`: the import of `os`, removed.

### step-05.7 a shorter line, by hand

A long message is a choice of words, so a person has to shorten it. The new message is short enough for one line.

patch$ src/api-service/api/service.py

::: do
Apply the patch. It changes the value of `"message"` in the `return` at the end of the file.
:::

::: watch
The value of `"message"` in the `return` of `calculate_euclidean_distance()` is now shorter than 120 characters.
:::

$ just lint

ruff prints `All checks passed!`. Note that ruff never reports `unused_function()`. No rule of `E`, `W` or `F` looks for
a function that nothing calls.

files:
- diff `src/api-service/api/service.py`: a shorter message.
  show: `"message": "This is a very long line that exceeds 120 characters blah",`

### step-05.8 just check runs every check

The layout, the rules and the tests each have a recipe now. The recipe `check` runs all three, and stops at the
first that fails.

patch$ justfile

::: do
Read the recipe `check` at the end of the `justfile`.
:::

::: watch
The recipe `check`, at the end of the `justfile`, runs `just fmt --check`, `just lint` and `just test -q`, in that
order.
:::

$ just check

just prints each command, and the last line is `21 passed`.

files:
- diff `justfile`: the recipe `check`.
  show: `just fmt --check`

## step-06 Checks before every commit

$ just setup

`just check` works only when someone runs it. pre-commit runs checks before every `git commit`, and stops the commit
when a check fails. The checks are in `.pre-commit-config.yaml`.

### step-06.1 pre-commit, in the group lint

pre-commit checks the code, and the server does not need it to run. So it goes into the group `lint`, with ruff.

do$ uv add --group lint pre-commit

$ uv run pre-commit --version

uv prints the version of pre-commit.

files:
- diff `pyproject.toml`: pre-commit, in the group `lint`.
  show: `"pre-commit>=4.7.0",`
- diff `uv.lock`: the exact versions of pre-commit, and of the packages that it needs.

### step-06.2 the checks that run before every commit

pre-commit needs a list of checks. `.pre-commit-config.yaml` names two repositories of checks. The first,
pre-commit-hooks, checks any file. The second, ruff-pre-commit, runs ruff's linter and formatter with the settings of
`pyproject.toml`.

patch$ .pre-commit-config.yaml

::: do
Apply the patch, and read the two entries under `repos`. Each has an address, a version after `rev`, and its checks
under `hooks`.
:::

::: watch
The new file `.pre-commit-config.yaml` names pre-commit-hooks at `v6.0.0`, with seven checks of any file, such as
`trailing-whitespace` and `detect-private-key`. Then it names ruff-pre-commit at `v0.17.1`, with `ruff-check` and
`ruff-format`.
:::

pre-commit checks the files that git tracks, the files of the last commit. The first run downloads the checks, which
takes a minute.

$ uv run pre-commit run --all-files

pre-commit prints a line for each check, `Passed` or `Skipped`.

::: do
Your new files are not in a commit yet. So `check yaml` and `ruff check` say `(no files to check)Skipped`, and they
check your files at your next `git commit`. `check json` says `Skipped` too, because the project has no JSON file.
:::

::: watch
Every file of the project is in a commit here. `check json` says `Skipped`, because the project has no JSON file.
Every other check says `Passed`.
:::

files:
- file `.pre-commit-config.yaml`: the checks that run before every commit.
  show: `- id: ruff-check`

### step-06.3 just hook installs the checks as a git hook

pre-commit runs before every commit only after it installs a hook in `.git/hooks/`. `just setup` never changes git,
so the hook has a recipe of its own, `hook`. You run it once in each clone.

patch$ justfile

::: do
Read the recipe `hook`, after the recipe `setup`.
:::

::: watch
The recipe `hook`, after the recipe `setup`, runs `uv run pre-commit install`.
:::

$ just hook

pre-commit prints `pre-commit installed at .git/hooks/pre-commit`. From now on, `git commit` runs the checks first.

files:
- diff `justfile`: the recipe `hook`.
  show: `uv run pre-commit install`

## step-07 The server in Docker

$ just setup

The server runs on this machine, with its `.venv` and this version of macOS or Linux. Production is another machine,
such as a server on Google Cloud. A Docker image holds the server with everything it needs, so it runs the same on
any machine with Docker. The run image is the image that production runs, our deployment.

### step-07.1 what docker build leaves out

`docker build` sends the files of the folder to Docker. The folder has `.venv`, the caches of the tools, git and maybe
secrets. None of them belongs in an image. `.dockerignore` lists them.

patch$ .dockerignore

::: do
Apply the patch, and read its list.
:::

::: watch
The new file `.dockerignore` lists `.venv/`, the caches of pytest, ruff and coverage, `.git/`, `secrets/` and `.env`.
:::

$ cat .dockerignore

`cat` prints the list.

files:
- file `.dockerignore`: what `docker build` never sends to Docker.
  show: `.venv/`

### step-07.2 the run image, the server as it runs in production

The run image needs Python, the dependencies of the server, and the code. It does not need uv, the tests, ruff or
pre-commit. The `Dockerfile` builds the image in stages, and the run image takes only what the server needs.

patch$ Dockerfile

::: do
Apply the patch, and read the three stages, which start at each `FROM`. Read the line `RUN uv sync` in the stage
`build`, and the line `COPY --from=build` and the `CMD` in the stage `run`.
:::

::: watch
The stage `uv` takes uv from its own image. The stage `build` installs the dependencies of `uv.lock` into
`/opt/venv`, with `--no-default-groups`, so no test tool and no ruff. The stage `run` copies `/opt/venv` and the
package `api`, and runs uvicorn as the user `app`, with four workers.
:::

$ docker build --target run --tag cheesecracker-run .

Docker prints each line of the `Dockerfile` as it runs, and the last line names the image `cheesecracker-run`.

$ docker image ls cheesecracker-run

The image is about 173 MB, nearly all of it Python and the dependencies of the server.

files:
- file `Dockerfile`: the stages `uv`, `build` and `run`.
  show: `RUN uv sync --locked --no-default-groups`

### step-07.3 just docker-build and just docker-serve

Two recipes give the Docker commands short names. `docker-build` builds an image of the `Dockerfile`, and
`docker-serve` runs the server in the run image at port 9000.

patch$ justfile

::: do
Read the recipes `docker-build` and `docker-serve` at the end of the `justfile`.
:::

::: watch
The recipe `docker-build` takes the name of a stage, and runs `docker build --target` with it. `docker-serve` runs the
image `cheesecracker-run`, and publishes its port 9000 at port 9000 of this machine. `--rm` removes the container
when it stops.
:::

$ just docker-build run

The build takes a second, because Docker has every layer in its cache.

The server of `just serve` uses port 9000. Stop it with Ctrl-C in its terminal. Then run the server in Docker, in that
terminal.

runs$ just docker-serve

uvicorn prints `Uvicorn running on http://0.0.0.0:9000`, and starts four workers. In another terminal, ask it for its
health.

$ curl -s localhost:9000/health

The server in Docker answers `{"status":"healthy"}`, as the server on this machine did.

files:
- diff `justfile`: the recipes `docker-build` and `docker-serve`.
  show: `docker run --rm --name cheesecracker-run -p 9000:9000 cheesecracker-run`

## step-08 System tests

$ just setup

The integration tests call the app in the same process as the tests. They cannot tell whether the image starts, or
whether the server answers on its port. A system test sends real HTTP requests to the running server, as the Cheese
App does. Here the server is the run image, our deployment.

### step-08.1 requests, for tests that send real HTTP requests

The system tests send their requests with the package requests. It is a tool of the tests, so it goes into the group
`test`.

do$ uv add --group test requests

$ uv run python -c "import requests; print(requests.__version__)"

Python prints the version of requests.

files:
- diff `pyproject.toml`: requests, in the group `test`.
  show: `"requests>=2.34.2",`
- diff `uv.lock`: the exact versions of requests, and of the packages that it needs.

### step-08.2 system tests, against the server in Docker

Five system tests send requests to `localhost:9000`. Four of them need a server there, and skip when none answers. The
fifth imports the app, and needs no server.

patch$ tests/system/test_system_api.py

::: do
Make the folder `tests/system/`, and apply the patch. Read the function `is_api_running()`, then the line
`@pytest.mark.skipif` above the class `TestAPIEndpoints`.
:::

::: watch
The new file `tests/system/test_system_api.py` sets `API_BASE_URL` to `http://localhost:9000`. `is_api_running()` asks
for `/health`. The decorator `@pytest.mark.skipif` skips the class `TestAPIEndpoints` when no server answers. The
class `TestAPIWithoutServer` imports the app.
:::

The server of `just docker-serve` must still run. Run the system tests against it.

$ uv run pytest tests/system -v

pytest prints `5 passed`. The four tests of `TestAPIEndpoints` sent their requests to the server in Docker.

files:
- file `tests/system/test_system_api.py`: five system tests, through real HTTP requests to `localhost:9000`.
  show: `@pytest.mark.skipif(not is_api_running(), reason="API not running at localhost:9000")`

## step-09 The dev image

$ just setup

The checks run on this machine. Another person, or a CI service, runs them on another system, with other versions of
the tools. The dev image holds the whole project with every group of dependencies, and uv, just and git. So the
checks run in the same Linux, anywhere Docker runs.

### step-09.1 the dev image, to work on the project in Linux

The dev image is a fourth stage of the `Dockerfile`, `dev`. It takes uv from the stage `uv`, and nothing from the
run image.

patch$ Dockerfile

::: do
Apply the patch. Read the new lines at the top of the file, then the stage `dev` at the end. Read its line `RUN uv
tool install`, and its line `RUN uv sync --locked`, which installs every group.
:::

::: watch
The stage `dev` installs git and curl, copies uv, and installs just as a package of PyPI. It installs every group of
`uv.lock` into `/opt/venv`, copies the project into `/work`, and starts a shell. `/opt/venv` is outside `/work`, so a
`.venv` of this machine in a mounted folder is never used.
:::

$ docker build --target dev --tag cheesecracker-dev .

The last line names the image `cheesecracker-dev`.

$ docker image ls cheesecracker-dev

The image is about 353 MB, twice the run image, because it has every tool.

files:
- diff `Dockerfile`: the stage `dev`, and its lines in the comment at the top.
  show: `FROM python:3.12-slim-bookworm AS dev`

### step-09.2 just docker-shell and just docker-test

Two recipes run the dev image with this folder mounted at `/work`. An edit here shows in there at once, so the image
needs no rebuild after a change of the code. `docker-shell` opens a shell in the image. `docker-test` runs the recipe
that you name in the image, or `just check` if you name none.

patch$ justfile

::: do
Read the recipes `docker-shell` and `docker-test` at the end of the `justfile`.
:::

::: watch
`docker-shell` runs `docker run --rm -it -v "$PWD":/work cheesecracker-dev`. `-v` mounts this folder at `/work`, and
`-it` gives the shell a terminal. `docker-test` runs `just` in the image, with its words, or with `check`.
:::

$ just docker-test

`just check` runs in Linux. The last line is `22 passed, 4 skipped`. The four system tests skip, because inside the
container `localhost:9000` is the container itself, and no server answers there.

$ just docker-test test tests/unit -q

The eleven unit tests pass in Linux.

files:
- diff `justfile`: the recipes `docker-shell` and `docker-test`.
  show: `docker run --rm -v "$PWD":/work cheesecracker-dev just "${@:-check}"`

## step-10 A new endpoint

$ just setup

The Cheese App needs to add two numbers. The endpoint `/add/` takes `x` and `y`, and returns their sum. An integration
test calls it. Then every check of the project runs again, on this machine, in the dev image, and against a new run
image.

patch$ src/api-service/api/service.py
patch$ tests/integration/test_api.py

::: do
Apply both patches. Read the function `add_numbers()` at the end of `service.py`, and `test_add_endpoint()` at the end
of `test_api.py`.
:::

::: watch
`add_numbers()`, at the end of `service.py`, answers `/add/` with `{"result": x + y}`. `test_add_endpoint()`, at the
end of `test_api.py`, asks for `/add/?x=10&y=20`, and expects 30.
:::

$ just check

The checks pass on this machine. The integration tests are now eleven.

$ just docker-test

The same checks pass in the dev image.

The run image still has the old code, because an image is built once and does not change. Build it again, stop the old
server with Ctrl-C, and run the new one.

$ just docker-build run

runs$ just docker-serve

$ curl -s "localhost:9000/add/?x=10&y=20"

The server in Docker answers `{"result":30.0}`.

$ uv run pytest tests/system -q

The system tests print `5 passed` against the new run image.

files:
- diff `src/api-service/api/service.py`: the endpoint `/add/`.
  show: `return {"result": x + y}`
- diff `tests/integration/test_api.py`: the test of `/add/`.
  show: `response = client.get("/add/?x=10&y=20")`
