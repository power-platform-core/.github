# Contributing to Power Platform Core

Thank you for your interest in contributing to Power Platform Core! We welcome contributions of all kinds — bug fixes, new features, documentation improvements, and more.

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [Getting Started](#getting-started)
- [How to Contribute](#how-to-contribute)
- [Development Workflow](#development-workflow)
- [Commit Message Guidelines](#commit-message-guidelines)
- [Pull Request Process](#pull-request-process)

---

## Code of Conduct

By participating in this project, you agree to abide by our [Code of Conduct](CODE_OF_CONDUCT.md). Please read it before contributing.

---

## Getting Started

1. **Fork** the relevant repository in the [`power-platform-core`](https://github.com/power-platform-core) organisation.
2. **Clone** your fork locally:
   ```bash
   git clone https://github.com/<your-username>/<repo-name>.git
   cd <repo-name>
   ```
3. **Install dependencies** (each repository includes setup instructions in its own README).
4. **Create a feature branch** from `main`:
   ```bash
   git checkout -b feature/my-new-feature
   ```

---

## How to Contribute

### Reporting Bugs

- Search [existing issues](../../issues) to avoid duplicates.
- Open a new issue using the **Bug Report** template and include:
  - A clear description of the problem
  - Steps to reproduce
  - Expected vs. actual behaviour
  - Environment details (OS, Python version, Docker version, etc.)

### Requesting Features

- Search [existing issues](../../issues) first.
- Open a new issue using the **Feature Request** template.
- Describe the problem the feature solves, not just the implementation.

### Contributing Code

- Start with an issue. If one doesn't exist, open one so we can discuss the approach before you invest time writing code.
- Keep changes focused — one feature or bug fix per pull request.

---

## Development Workflow

### Prerequisites

- Python 3.11+
- Docker & Docker Compose
- PostgreSQL client (`psql`)
- Redis CLI

### Local Setup

```bash
# Copy environment variables
cp .env.example .env

# Start infrastructure services
docker compose up -d postgres redis

# Install Python dependencies
pip install -r requirements.txt

# Run database migrations
alembic upgrade head

# Start the service
uvicorn app.main:app --reload
```

### Running Tests

```bash
pytest -v
```

### Linting & Formatting

```bash
ruff check .
ruff format .
```

---

## Commit Message Guidelines

We follow the [Conventional Commits](https://www.conventionalcommits.org/) specification:

```
<type>(<scope>): <short description>
```

**Types:** `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `chore`

**Examples:**
```
feat(auth): add JWT refresh token support
fix(db): handle connection pool exhaustion gracefully
docs(readme): update local setup instructions
```

---

## Pull Request Process

1. Ensure all tests pass locally.
2. Update documentation if your change affects public behaviour.
3. Fill in the pull request template completely.
4. Request a review from at least one maintainer.
5. Address all review comments before the PR is merged.
6. PRs are merged using **squash and merge** to keep a clean history.

---

## Questions?

Feel free to open a [Discussion](../../discussions) or reach out to a maintainer via an issue comment.
