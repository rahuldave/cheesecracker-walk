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

## What docker build leaves out

$ just setup

Our first file for Docker, `.dockerignore`, lists the files that never go into an image.

{{include step-07.1}}

## The run image

$ just setup

The run image holds Python, the dependencies of the server and its code, and starts the server. Three recipes build
it, start it and stop it.

{{include step-07.2..step-07.3}}

## System tests, against the run image

A system test sends real HTTP requests to a running server, as a user of the app does. The run image is our
deployment, so the system tests send their requests to the server in it.

{{include step-08}}

## The dev image

The dev image runs the checks in Linux, anywhere Docker runs.

{{include step-09}}

## A change, through both images

A change of the code goes through both images. The dev image checks it, and the run image has to be built again.

{{include step-10}}
