We build the tests of cheesecracker, the API server of the Cheese App. A test runs the code and checks what it gives
back, so a later change cannot break the server without a failure. [pytest](https://docs.pytest.org/) runs the tests. It finds the
test functions of a project, runs them, and reports each failure. This tutorial takes the parts about tests out of
the whole tutorial of cheesecracker. It leaves out ruff, pre-commit and the dev image.

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

The tests come in three kinds, from the bottom of the testing pyramid to its top:

| Kind | What it calls | What it needs |
|---|---|---|
| Unit | One function alone, `power()` | Nothing |
| Integration | The app, through FastAPI's `TestClient`, in the same process | No server |
| System | The running server, over HTTP | A server at port 9000, the run image in Docker |

Some changes of the whole tutorial are not in this one. After such a gap, the next section says what those changes
did, and starts from the code that they left. It first runs `just setup`, which installs the packages of that code.

## The project

The project starts empty, with a `pyproject.toml`, a lockfile and a `justfile`. The next sections give it code and
tests.

{{include step-00}}

## A function, and its unit tests

A unit test calls one function alone. The first function of the server, `power()`, gets eleven of them.

{{include step-01}}

## The server under test

The integration tests and the system tests need a server to call. This section writes it, with two answers, `/` and
`/health`, and the recipes `just serve` and `just stop`.

{{include step-02}}

## Integration tests

An integration test sends a request to the app, with no server and no network. First the app gets a new request
that does some work. `/euclidean_distance/` answers with the distance of a point from the origin.

{{include step-03}}

## Coverage

The tests pass, but nothing says which lines of the code they run. Coverage counts them.

{{include step-04}}

## The run image, a server to test against

The whole tutorial then makes a change in a hurry. It adds an import that nothing uses, a function
`unused_function()` that nothing calls, and a long `message` in the answer of `/euclidean_distance/`. Then ruff, a
tool that formats code and checks it, fixes the spaces and blank lines, and removes the import. A person shortens the
message. pre-commit then runs the checks before each commit. The tests pass all along.

A system test needs a server that runs as it will run in production. Docker builds that server as an image, the run
image.

{{include step-07}}

## System tests, against the run image

The system tests send real HTTP requests to the run image. They need a server, so they run apart from the other
tests, and fail when no server answers.

{{include step-08}}
