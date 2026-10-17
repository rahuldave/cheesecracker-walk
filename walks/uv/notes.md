We follow the dependencies of cheesecracker, the API server of the Cheese App. [uv](https://docs.astral.sh/uv/) is
a tool that manages a Python project, written in Rust by Astral. It installs Python and the packages of the project
into the folder `.venv`, and writes their exact versions into one lockfile, `uv.lock`. So every machine gets the same
versions. `uv add` adds a package, and `uv run` runs a command with the project's Python.

The packages come in groups. The server
needs some of them to run. The tests and the checks need the others. This tutorial takes these parts out of the whole
tutorial of cheesecracker, and ends with the two Docker images, which install different groups.

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

Some changes of the whole tutorial are not in this one. After such a gap, the next section says what those changes
did, and starts from the code that they left. It first runs `just setup`, which installs the packages of that code.

## The project, and its `.venv`

The project starts as a `pyproject.toml`, a `.python-version` and a lockfile. `just setup` makes the environment,
`.venv`, from the lockfile.

{{include step-00}}

## A group for the tests

$ just setup

The whole tutorial first writes the function `power()`, in `src/api-service/api/utils.py`. Its tests need pytest, and
the server never does. So pytest goes into a group of its own.

{{include step-01.2}}

## The dependencies of the server

$ just setup

The whole tutorial then tells pytest where the tests are, adds eleven unit tests of `power()`, and the recipe
`just test`. Next comes the server, which needs FastAPI to build it and uvicorn to run it. They are dependencies of the project itself, and of no
group.

{{include step-02.1}}

## More tools for the tests

$ just setup

The whole tutorial then writes the app, the recipes `just serve` and `just stop`, and an endpoint that computes a
distance. The integration tests need two more packages, and they go into the group `test`.

{{include step-03.2}}

## One more tool for the tests

$ just setup

The whole tutorial then adds ten integration tests. The package pytest-cov measures which lines of the code the tests
run, and it goes into the group `test`.

{{include step-04.1}}

## A second group, for the checks

$ just setup

The whole tutorial then adds the recipe `just cov`, and a change made in a hurry. ruff checks the code, and the server
does not need it. So ruff starts a second group, `lint`.

{{include step-05.2}}

## pre-commit, in the group lint

$ just setup

The whole tutorial then sets up ruff, and fixes the code with it. pre-commit runs the checks before every commit, so
it joins ruff in the group `lint`.

{{include step-06.1}}

## The run image installs no group

$ just setup

The whole tutorial then writes the checks of pre-commit, and `.dockerignore`. The run image is the server as it runs
in production. It needs the dependencies of the project, and no group.

{{include step-07.2}}

## One more tool for the tests, after the run image

$ just setup

The whole tutorial then adds recipes for Docker. The system tests send real HTTP requests, with the package `requests`.
`requests` goes into the group `test`, and the run image never sees it.

{{include step-08.1}}

## The dev image installs every group

$ just setup

The whole tutorial then adds the system tests, and the recipe `just test-system`. The dev image is for working on the
project, so it installs every group.

{{include step-09.1}}
