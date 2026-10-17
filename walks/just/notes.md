We follow the `justfile` of cheesecracker, the API server of the Cheese App. The program [just](https://just.systems/)
reads this file. The
file gives each task of the project a short name, and each named task is a recipe.

The recipes keep the commands in one place, so a person, a git hook and Docker run the same
thing. `just` alone lists them, and `just --show NAME` prints one. This tutorial takes the recipes out of the whole
tutorial of cheesecracker, one at a time, as each one comes in. Every recipe here is in the `justfile` of the project
cheesecracker, in the folder of its code.

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
