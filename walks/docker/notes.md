We put cheesecracker, the API server of the Cheese App, into Docker. A Docker image holds a small Linux system with
the server and everything it needs, so it runs the same on any machine with Docker. We build two images from one
lockfile. The run image is the server as it runs for the
real users of the app, which we call production. It is our deployment.

The dev image holds the project with every tool. This tutorial is the part of the whole tutorial of cheesecracker that adds Docker.

The images depend on the groups of dependencies of the project. The group `test` holds the tools of the tests, and the
group `lint` holds ruff and pre-commit. The run image installs no group, and the dev image installs every group. The
tutorial uv, beside this one, shows how the groups came to be.

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
