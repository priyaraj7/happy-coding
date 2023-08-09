# Installing Private Packages from Artifactory in Your Docker Project

## Introduction:

When working on projects that require private packages, developers often rely on package managers to fetch dependencies from private repositories like Artifactory. Recently, I encountered a similar scenario where I needed to install a private package from Artifactory for my project. While the installation worked seamlessly in my local environment, I faced challenges when trying to automate the process using Jenkins. After some troubleshooting with the guidance of a senior developer, I identified the issue and found a solution. In this blog post, I'll share the steps I took to successfully install private packages from Artifactory within a Docker container.

## Identifying the Issue:

At first, I attempted to install the private package directly from Artifactory using Poetry, the Python package manager. My `pyproject.toml` file included two source URLs for the same package - one for the private repository and another for a public repository. This redundancy caused Jenkins' job to fail as it was getting confused about which source to use.

To provide a clear view, here is how the relevant part of my `pyproject.toml` file looked initially:

```toml
[[tool.poetry.source]]
name = "artifactory"
url = "https://url/to/artifactory_repo"
priority = "default"

[[tool.poetry.source]]
name = "private"
url = "https://url/to/artifactory_private_repo"
secondary = "primary"
```

## The Solution:

With the assistance of a senior developer, I realized that the Artifactory private repository already contained all the packages from the public repository. To resolve the issue, I manually deleted both source URLs for the package in the `pyproject.toml` file.

## Pulling the Private Package in Docker:

Since my project utilized Docker, I needed to ensure that the private package was installed within the Docker container. Here are the steps I followed:

1. Configure the Package Source as a Supplemental Package Source to Your Project:

To fetch packages from the private Artifactory repository, I added it as a supplemental package source using the following Poetry command:

```bash
poetry source add --priority=supplemental artifactory https://artifactory.private.example.org/simple/
```

2. Authenticate into Your Private Repository:

To authenticate and access the private repository, I configured the repository credentials using the following command:

```bash
poetry config http-basic.artifactory <username> <password>
```

3. Search and Install the Private Package:

With the configuration complete, I could easily search for the desired private package using Poetry's search command:

```bash
poetry search <private package name>
```

And to install the package, I used the following command:

```bash
poetry add <private package name>
```

## Building the Docker Image:

Next, I built the Docker image, incorporating the required authentication credentials as build-time arguments. Here's a snippet of the Dockerfile:

```Dockerfile
FROM python:3.9.6-alpine

# two build-time arguments used later in the build process to configure authentication for package downloads from the repository.
ARG ARTIFACTORY_CREDS_USR
ARG ARTIFACTORY_CREDS_PSW

WORKDIR /app

RUN pip install poetry

COPY pyproject.toml poetry.lock /app/
#configures Poetry to use HTTP basic authentication when interacting with the specified Artifactory repository.
RUN poetry config http-basic.artifactory $ARTIFACTORY_CREDS_USR $ARTIFACTORY_CREDS_PSW

RUN poetry config virtualenvs.create false && poetry install --no-interaction --no-ansi

# Continue building the image...
```

Here's how Docker build command should look like:

```bash
docker build --build-arg ARTIFACTORY_CREDS_USR=<your username> --build-arg ARTIFACTORY_CREDS_PSW=<your password> -t <image-name> .
```

The `--build-arg` flag allows you to pass build-time arguments to the Docker build process, which will be used later in the build context (in this case, in the Dockerfile).

With this command, Docker will start building the image based on the instructions in the Dockerfile, and it will also set the environment variables `ARTIFACTORY_CREDS_USR` and `ARTIFACTORY_CREDS_PSW` during the build process. These variables will be used by Poetry to authenticate and fetch the private package from Artifactory.

## Conclusion:

By following these steps, I successfully resolved the issue with my Jenkins job failing to install the private package from Artifactory. The key was to remove redundant source URLs from the `pyproject.toml` file and ensure the correct configuration of the private repository as a supplemental source. Additionally, configuring the repository credentials allowed seamless access to the private packages during the Docker build process. Happy coding!

## Credit:

1. [Private Repository Example](https://python-poetry.org/docs/repositories/#private-repository-example)
