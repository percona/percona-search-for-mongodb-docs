# Percona Search for MongoDB documentation

Welcome to Percona Search for MongoDB documentation!

## Overview

Percona Search for MongoDB adds full-text and vector search to self-managed Percona Server for MongoDB deployments. Search runs in a separate process called `mongot`, which maintains Apache Lucene-based search indexes and executes queries submitted through the `$search`, `$searchMeta`, and `$vectorSearch` aggregation stages.

The Percona Search for MongoDB code is [here](https://github.com/percona/percona-mongot).

This repository contains the source files for [Percona Search for MongoDB documentation](https://docs.percona.com/percona-search-for-mongodb/). The documentation is written in [Markdown](https://www.markdownguide.org/basic-syntax/) and is built with [MkDocs](https://www.mkdocs.org/) and the [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/) theme.

## Build the documentation locally

The steps below work on macOS and Linux.

### Prerequisites

- [Git](https://git-scm.com/)
- Python 3.9 or later with the `venv` module

**macOS** (using [Homebrew](https://brew.sh/)):

```sh
brew install python git
```

> **Note:** Don't use the Homebrew `mkdocs` formula (`brew install mkdocs`). It runs in its own isolated Python environment and can't see the theme and plugins this project needs, which results in errors such as `No module named 'material'`. If you have it installed, either remove it with `brew uninstall mkdocs` or always run MkDocs from the virtual environment described below.

**Linux** (Debian/Ubuntu):

```sh
sudo apt update
sudo apt install python3 python3-venv python3-pip git
```

**Linux** (RHEL/Fedora/Oracle Linux):

```sh
sudo dnf install python3 python3-pip git
```

### 1. Clone the repository

```sh
git clone https://github.com/percona/percona-search-for-mongodb-docs.git
cd percona-search-for-mongodb-docs
```

### 2. Create and activate a virtual environment

```sh
python3 -m venv .venv
source .venv/bin/activate
```

Your shell prompt now starts with `(.venv)`. Activate the environment again in every new terminal session before running MkDocs. To leave it, run `deactivate`.

### 3. Install the dependencies

```sh
pip install --upgrade pip
pip install -r requirements.txt
```

### 4. Preview the documentation

```sh
mkdocs serve -f mkdocs.yml
```

Open <http://127.0.0.1:8000/percona-search-for-mongodb/> in your browser. MkDocs rebuilds the site and reloads the page whenever you save changes in the `docs` directory or in `mkdocs.yml`. Press `Ctrl+C` to stop the server.

> **Note:** The `GET /versions.json HTTP/1.1" code 404` warnings in the server log are expected. The version selector reads this file, which is generated only when the documentation is published with `mike`.

### 5. Build the static site

```sh
mkdocs build -f mkdocs.yml
```

The generated HTML files are placed in the `site` directory.

### Troubleshooting

- **`No module named 'material'` or another missing plugin**: MkDocs isn't running from the virtual environment. Activate it with `source .venv/bin/activate` and check that `which mkdocs` points to `.venv/bin/mkdocs`.
- **`has no git logs, using current timestamp`**: The page hasn't been committed yet. The warning disappears after you commit the file.

## Contributing

We welcome all contributors. For how to contribute to documentation, read the [Contributing guide](CONTRIBUTING.md).

## Licensing

Percona is dedicated to **keeping open source open**. Wherever possible, we strive to include permissive licensing for both our software and documentation. This documentation is licensed under the [Creative Commons Attribution 4.0 International](LICENSE) license.
