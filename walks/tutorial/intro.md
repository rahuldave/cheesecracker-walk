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
