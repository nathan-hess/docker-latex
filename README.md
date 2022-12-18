# LaTeX in Docker

### Docker images with dependencies for building PDFs with LaTeX

This repository contains files configured to build Docker images with a relatively comprehensive set of TeX dependencies installed.  The main intention of such images is to support automated workflows (such as GitHub Actions) that build LaTeX documents, or to provide a way to quickly and easily build LaTeX documents on a personal computer without the need to install and configure a TeX distribution.


## Image Description

The Docker image contains an installation of [TeX Live](https://www.tug.org/texlive/) (with all packages installed), as well as various other relevant software for building PDFs and documentation (Graphviz, Python, etc.).

### Docker Hub repository: https://hub.docker.com/r/nathanhess/latex

| Tag    | Base Image                                | Build Context | Default User |  
|:-------|:------------------------------------------|:--------------|:-------------|
| latest | [Ubuntu](https://hub.docker.com/_/ubuntu) | `dockerfile/` | root         |

The packages in the image are updated and the image is published to Docker Hub twice a month using GitHub Actions workflows.  All configuration files and workflow logs are available through GitHub.

### GitHub repository: https://github.com/nathan-hess/docker-latex


---

## Background

One of the challenges of creating TeX documents is that any given project typically requires a number of [packages](https://ctan.org/pkg).  It can be tedious and time-consuming to install and update these packages, and some users (for security, disk capacity, or device performance reasons) may not want to install packages they only need for a one-off project on their system.

This repository contains a Docker image that attempts to address this challenge: it configures an image with a relatively comprehensive set of TeX packages, eliminating the need for users to install individual packages.  Furthermore, because the packages are installed *within* a Docker image, end users don't need to install any TeX packages on their local system -- all installation happens in an isolated environment inside the image.

Building a Docker image with a reasonably comprehensive set of TeX packages can take considerable time.  For this reason, GitHub Actions is configured in this repository to build and publish the image to Docker Hub.  End users can then simply download and run the image from Docker Hub.  For GitHub Actions runners, this reduces the setup time from ~10 minutes to ~2 minutes.


## Usage

### GitHub Actions Workflows

One potential use case of this image is to build PDFs as part of GitHub Actions workflows or similar CI/CD pipelines.

An example GitHub Actions workflow is shown below.  This workflow builds a PDF from a file `main.tex` located in the root of the repository and uploads the resulting PDF as a GitHub Actions artifact.

```yaml
name: Build and Upload PDF

on:
  pull_request:
    types: [opened, reopened, synchronize]
  push:
    branches: main
  workflow_dispatch:

jobs:
  build:
    name: Build PDF and Upload as GitHub Actions Artifact
    runs-on: ubuntu-latest
    container:
      image: nathanhess/latex:latest
    steps:
      - name: Check Out Repository Files
        uses: actions/checkout@v3

      - name: Build PDF
        run: |
          pdflatex -synctex=1 -interaction=nonstopmode "main.tex"
          bibtex "main.aux"
          pdflatex -synctex=1 -interaction=nonstopmode "main.tex"
          pdflatex -synctex=1 -interaction=nonstopmode "main.tex"

      - name: Upload PDF as Artifact
        uses: actions/upload-artifact@v3
        with:
          name: latex-pdf
          path: main.pdf
          retention-days: 30
          if-no-files-found: error
```

This or a similar workflow could be useful in cases such as a team project, as it verifies that the project report can be successfully built each time team members propose or merge changes to the repository, and the uploaded artifacts serve as a convenient "preview" of the report.  The workflow could even be expanded to automatically email the report to customers on a schedule or on merges to `main`.


### Building PDFs Locally

If you'd like use this image to build a LaTeX document without installing TeX packages directly on your system, the quickest and easiest approach is to download the image from Docker Hub:

```
C:\>docker pull nathanhess/latex:latest
```

Alternatively, you can also clone this repository and [build the image](https://docs.docker.com/engine/reference/commandline/build/) from the Dockerfile in `dockerfile/Dockerfile`.

Next, create a container from the image, creating a [bind mount](https://docs.docker.com/storage/bind-mounts/) with the `-v` flag to allow your project's TeX files to be accessed by the container (replace `C:\Users\%USERNAME%\myProject` with the path to your project): 

```
C:\>docker run --rm -it -v "C:\Users\%USERNAME%\myProject":/workspaces nathanhess/latex
```

Finally, run any commands inside the image necessary to build your LaTeX document:

```
root@hostname$ cd /workspaces
root@hostname:/workspaces$ pdflatex -synctex=1 -interaction=nonstopmode "main.tex"
root@hostname:/workspaces$ exit
C:\>
```

> **Note** 
> Using a bind mount is the simplest approach, but it will increase PDF build times.  Better performance may be achieved by copying your project's TeX files to the container's filesystem or using a [Docker volume](https://docs.docker.com/storage/volumes/), althought these options are more complex to configure.


## References

- [Docker](https://www.docker.com/)
  - [Getting Started Overview](https://docs.docker.com/get-started/overview/)
  - [Command-Line Reference](https://docs.docker.com/engine/reference/commandline/docker/)
  - [Dockerfile Reference](https://docs.docker.com/engine/reference/builder/)
- [GitHub Actions](https://docs.github.com/en/actions)
  - [Running jobs in a container](https://docs.github.com/en/actions/using-jobs/running-jobs-in-a-container)
- [TeX Live](https://www.tug.org/texlive/)
