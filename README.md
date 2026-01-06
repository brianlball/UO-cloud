# urbanopt-cloud

Docker container for URBANopt CLI 1.1.0 on Ubuntu 22.04.

## Overview

This repository contains a Dockerfile that builds a Docker container with URBANopt CLI 1.1.0 installed on Ubuntu 22.04. The container is automatically built, tested, and pushed to Docker Hub via GitHub Actions.

## Features

- **URBANopt CLI 1.1.0**: Pre-installed and configured
- **Ubuntu 22.04**: Base image with all required dependencies
- **Automated CI/CD**: GitHub Actions workflow for building, testing, and publishing
- **Versioning**: Automatic semantic versioning via Git tags

## Usage

### Pull from Docker Hub

```bash
docker pull brianlball/urbanopt-cloud:latest
```

### Run the Container

```bash
# Run with default command (shows version)
docker run --rm brianlball/urbanopt-cloud:latest

# Run interactively
docker run -it --rm brianlball/urbanopt-cloud:latest bash

# Run with a mounted workspace
docker run -it --rm -v $(pwd):/work brianlball/urbanopt-cloud:latest
```

### Build Locally

```bash
docker build -t urbanopt-cloud:latest .
```

## Running on the Cloud

### Option 1 - Running on an instance in the cloud (Docker container)

For a single-server cloud deployment (e.g., an AWS EC2 instance), the simplest and most reproducible approach is to run URBANopt inside a Docker container. This avoids installing URBANopt directly on the VM and makes upgrades/rollbacks as simple as changing the container tag.

> **URBANopt CLI commands and workflows are unchanged.** You will still use `uo create`, `uo run`, `uo process`, etc.—you’ll just run them inside the container.

#### 1. Provision a VM and install Docker
Create a cloud VM (e.g., Ubuntu 22.04 on AWS EC2) and install Docker. Ensure your user can run Docker commands.

#### 2. Pull the prebuilt URBANopt container
Pull a pinned version tag for reproducibility:

    docker pull brianlball/urbanopt-cloud:1.1.0

#### 3. Put your URBANopt project on the VM
Copy your project directory onto the VM (e.g., via SCP/rsync, or by downloading from S3).
Assume your project is located at:

- `/home/ubuntu/my-uo-project`

#### 4. Run URBANopt with the project mounted into the container
Start an interactive shell with your project mounted at `/work`:

    docker run --rm -it \
      -v /home/ubuntu/my-uo-project:/work \
      brianlball/urbanopt-cloud:1.1.0 \
      bash

Inside the container, your files are available at `/work`:

    cd /work
    uo --version

From here, run the same URBANopt CLI commands you would run on a normal Linux installation.

---

## Example workflow: create and run the example project (inside the container)

If you want to validate your environment end-to-end, you can create the URBANopt example project and run it:

    cd /work
    uo create --project-folder example_project
    uo create --scenario-file example_project/example_project.json
    uo run --feature example_project/example_project.json --scenario example_project/baseline_scenario.csv

The URBANopt run directory is created under the project directory.

---

## Example workflow: run *your existing* URBANopt project (inside the container)

If you already have a URBANopt project folder, mount it to `/work` and run commands against paths in `/work`.

Example (adjust paths/filenames to your project):

    cd /work
    uo run --feature path/to/feature_file.json --scenario path/to/scenario_file.csv

See the URBANopt “Getting Started” and workflow docs for the expected project structure and run outputs. :contentReference[oaicite:3]{index=3}

---

## Non-interactive usage (recommended for automation)

You can run URBANopt commands without opening an interactive shell by passing the command directly:

    docker run --rm \
      -v /home/ubuntu/my-uo-project:/work \
      brianlball/urbanopt-cloud:1.1.0 \
      uo --version

Or to run a simulation directly:

    docker run --rm \
      -v /home/ubuntu/my-uo-project:/work \
      brianlball/urbanopt-cloud:1.1.0 \
      uo run --feature /work/path/to/feature_file.json --scenario /work/path/to/scenario_file.csv

---

#### Notes
- Pin the container tag (e.g., `:1.1.0`) for reproducibility.
- For parallel scaling across many projects/scenarios, consider orchestration options (e.g., AWS Batch/ECS/EKS) or OpenStudio-server.




## GitHub Actions Workflow

The repository includes a GitHub Actions workflow that:

1. **Builds** the Docker container
2. **Tests** the container by running `uo --version`
3. **Pushes** to Docker Hub with appropriate tags:
   - `latest` - for commits to main/master branch
   - `<branch>` - for branch commits
   - `v1.0.0`, `v1.0`, `v1` - for semantic version tags
   - `<branch>-<sha>` - for commit SHA references

### Docker Hub Configuration

To enable automatic pushing to Docker Hub, configure the following repository secrets:

1. Go to your repository **Settings** → **Secrets and variables** → **Actions**
2. Click **"New repository secret"**
3. Add the following secrets:
   - `DOCKER_USERNAME`: Your Docker Hub username
   - `DOCKER_PASSWORD`: Your Docker Hub password or access token

### Versioning

To create a new versioned release:

```bash
git tag -a v1.0.0 -m "Release version 1.0.0"
git push origin v1.0.0
```

## Architecture Support

The Dockerfile supports both x86_64 (default) and arm64 architectures. To build for a specific architecture, use the `UO_ARCH` build argument:

```bash
# Build for x86_64 (default)
docker build -t urbanopt-cloud:x86_64 .

# Build for arm64
docker build --build-arg UO_ARCH=arm64 -t urbanopt-cloud:arm64 .
```

## License

See the URBANopt CLI license for details.
