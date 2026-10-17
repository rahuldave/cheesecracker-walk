We check the code of cheesecracker, the API server of the Cheese App. Code that works can still be hard to read, and
a reader can miss a bug in it. A formatter writes the code in one layout, and a linter checks it against rules.
[ruff](https://docs.astral.sh/ruff/) is both, in one fast tool written in Rust by Astral. It takes the place of black,
the usual formatter, and of flake8, the usual linter. [pre-commit](https://pre-commit.com/) then runs the checks before
every `git commit`, and stops the commit when one fails.

This tutorial takes these parts out of the whole
tutorial of cheesecracker.

The code is in the repository [rahuldave/cheesecracker](https://github.com/rahuldave/cheesecracker) on GitHub. When the
project is done, the server's code is in `src/api-service/api/`, and its tests are in `tests/unit/`, `tests/integration/` and
`tests/system/`. `pyproject.toml` and `uv.lock` list the Python packages that the project needs. `justfile` holds its
recipes, short names for the commands of the project. `Dockerfile` holds the instructions that build its two Docker images,
the run image of the server and the dev image with every tool.

The repository keeps every saved version of the project, in the order of the whole tutorial of cheesecracker. That tutorial has many
parts, and this tutorial follows some of them.

Each page covers one part of the tutorial. The top of the page links to the code as it is when the
part starts. The bottom links to the code as it is when the part ends.

We make the project in moves. A move is one small change, made by one patch or by one command. You make each move,
and the page checks your files against it.

The first sections build the code to check, a function, a server and their tests. Some changes of the whole tutorial are not in this one. After such a gap, the next section says what those changes
did, and starts from the code that they left. It first runs `just setup`, which installs the packages of that code.

## The project

The project starts empty, with a `pyproject.toml`, a lockfile and a `justfile`.

{{include step-00}}

## A function, and its unit tests

The first code to check is the function `power()`, with its unit tests.

{{include step-01}}

## The server

The server answers two requests, `/` and `/health`.

{{include step-02}}

## The distance endpoint, and integration tests

The server gets a new request, `/euclidean_distance/`, which works out how far a point is from the origin. New tests
send that request to the app, and check the answer.

{{include step-03}}

## A change in a hurry

$ just setup

The whole tutorial then adds pytest-cov, a tool that shows which lines of the code the tests run, and a recipe,
`just cov`, that runs it. The code of the server does not change.

Then someone changes the code in a hurry. It still works, and it reads worse.

{{include step-05.1}}

## ruff, the formatter and the linter

$ just setup

ruff goes into a dependency group of its own, `lint`. Its settings allow lines of up to 120 characters, and check the
rules that flake8, an older linter, checks. Two recipes run it. `just fmt` runs the formatter, and `just lint` runs the
linter.

{{include step-05.2..step-05.4}}

## The fixes

$ just setup

The formatter fixes the layout, and the linter removes the import that nothing uses. A person shortens the long line.

{{include step-05.5..step-05.7}}

## One recipe for every check

$ just setup

The layout, the rules and the tests each have a recipe. One recipe, `just check`, runs them all.

{{include step-05.8}}

## Checks before every commit

`just check` works only when someone runs it. pre-commit runs checks before every `git commit`.

{{include step-06}}
