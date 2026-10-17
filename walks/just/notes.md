We follow the `justfile` of cheesecracker, the API server of the Cheese App.

[just](https://just.systems/) is a command runner. A project keeps its commands in a file named `justfile`, and gives
each one a short name. Each named command is a recipe. `just NAME` runs a recipe, `just` alone lists them, and
`just --show NAME` prints one.

A recipe can take more words after its name, as in `just test tests/unit`, and it can run several lines of
commands. The program `just` is small, and its authors wrote it in Rust. Older projects often use a program named `make` for the same job.

The recipes keep the long commands of the project in one place. Everyone on the project runs them by the same short
names. Later, the project also runs in [Docker](https://docs.docker.com/), which runs a program in a container, a
sealed copy of a small Linux system. The dev image of the project holds `just` and the other tools, so Docker runs
the same recipes. This tutorial takes the recipes out of the whole
tutorial of cheesecracker, one at a time, as each one comes in. Every recipe here is in the `justfile` of the project
cheesecracker, in the folder of its code.

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

## The project, and its justfile

The first recipe, `setup`, makes the environment of the project.

{{include step-00}}

## just test

$ just setup

The whole tutorial first writes the function `power()`, and adds [pytest](https://docs.pytest.org/), a program that
runs tests. It tells pytest where the tests are, and writes eleven unit tests. The command that runs them gets a recipe.

{{include step-01.5}}

## just serve and just stop

$ just setup

The whole tutorial then adds [FastAPI](https://fastapi.tiangolo.com/), a library for web APIs, and writes the app
with it. It also adds [uvicorn](https://uvicorn.dev/), the server that runs the app. Two recipes start the server and stop it.

{{include step-02.3}}

## just cov

$ just setup

The whole tutorial then adds an endpoint, ten integration tests, and [pytest-cov](https://pytest-cov.readthedocs.io/), which measures
the lines that the tests run. Coverage gets a recipe.

{{include step-04.2}}

## just fmt and just lint

$ just setup

The whole tutorial then makes a change in a hurry. It adds [ruff](https://docs.astral.sh/ruff/), one tool that does two
jobs. As a formatter, it lays out the code in one fixed way. As a linter, it checks the code against rules. The
tutorial gives ruff the rules of flake8, an older linter. The formatter and the linter get a recipe each.

{{include step-05.4}}

## just check

$ just setup

The whole tutorial then fixes the code with ruff, and shortens a long line. One recipe runs every check.

{{include step-05.8}}

## just hook

$ just setup

The whole tutorial then adds [pre-commit](https://pre-commit.com/), a tool that runs checks before every commit. git runs them through a hook, a
file in git's own folder. `just setup` installs only Python and the packages, so the hook gets a recipe of its own.

{{include step-06.3}}

## The Docker recipes

$ just setup

[Docker](https://docs.docker.com/) packs the server, with Python and its packages, into an image. It builds the image
from a file called `Dockerfile`, and runs it as a container, a process kept apart from the rest of the machine. The
whole tutorial then writes `.dockerignore`, and a `Dockerfile` that makes the run image, the image that runs the
server. Three recipes build an image, start its server and stop it.

{{include step-07.3}}

## just test-system

$ just setup

$ just docker-build run

$ just docker-serve

The whole tutorial then adds the package [requests](https://requests.readthedocs.io/), which lets Python code send
HTTP requests, and writes system tests. A system test sends real HTTP requests to
the server at port 9000 of this machine. The system tests need a running server, so `just test` leaves them out, and
`just docker-serve` above starts one. The system tests get a recipe of their own, apart from `just check`.

{{include step-08.4}}

## just docker-shell and just docker-test

$ just setup

$ just docker-build dev

The whole tutorial then adds the dev image to the `Dockerfile`. Two recipes run it, with the folder of the project
mounted inside.

{{include step-09.2}}
