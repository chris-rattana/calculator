# Calculator — CI/CD pipeline (GitHub Actions + Trivy + Docker Hub)

A simple Python **calculator** package with **pytest** tests and a complete **CI/CD** pipeline:

- ✅ CI: matrix tests on **Python 3.8 / 3.9 / 3.10**, **flake8** lint, **Trivy** scan (SARIF to GitHub Security)
- ✅ CD: build & push Docker image to Docker Hub (**`chrisrattana/calculator:latest`**) when CI succeeds

---

## Project structure

```text
.
├── calculator/
│   ├── __init__.py
│   └── calculator.py
├── tests/
│   ├── __init__.py
│   └── test_calculator.py
├── Dockerfile
├── main.py
└── .github/workflows/
    ├── ci.yml
    └── cd.yml
```

---

## Run locally

### 1) Create a virtualenv (recommended)

```bash
python -m venv .venv
# Windows (Git Bash)
source .venv/Scripts/activate
# macOS/Linux
source .venv/bin/activate
```

### 2) Install test/lint tools

```bash
python -m pip install --upgrade pip
pip install pytest flake8
```

### 3) Run the demo entrypoint

```bash
python main.py
```

### 4) Run tests

```bash
pytest tests/
```

### 5) (Optional) Run lint

```bash
flake8 .
```

---

## CI pipeline (GitHub Actions)

Workflow: **`.github/workflows/ci.yml`**

Triggers:
- `push` on `main`
- `pull_request` to `main`
- manual run (`workflow_dispatch`)

Jobs:
- **test**: runs `flake8` + `pytest tests/` on Python **3.8 / 3.9 / 3.10**
- **trivy-scan**: runs Trivy (filesystem scan) and uploads results as **SARIF** to GitHub

Where to see Trivy results:
- GitHub repo → **Security** → **Code scanning alerts** (SARIF upload)

---

## CD pipeline (GitHub Actions → Docker Hub)

Workflow: **`.github/workflows/cd.yml`**

Trigger:
- `workflow_run` **after** CI workflow (**only if CI is successful**)

What it does:
- logs in to Docker Hub
- builds the Docker image
- pushes to: **`chrisrattana/calculator:latest`**

### Required GitHub Secrets

Repo → **Settings** → **Secrets and variables** → **Actions** → **New repository secret**

- `DOCKER_USERNAME` = your Docker Hub username (here: `chrisrattana`)
- `DOCKER_PASSWORD` = a Docker Hub **Access Token** (recommended, needs **Read & Write**)

---

## Docker usage

### Pull the published image

```bash
docker pull chrisrattana/calculator:latest
```

### Quick sanity check (example)

```bash
docker run --rm chrisrattana/calculator:latest python -c "from calculator.calculator import add; print(add(1, 2))"
```

### Build locally

```bash
docker build -t chrisrattana/calculator:local .
```

---

## Notes

- If the CD pipeline fails with auth errors, verify:
  - `DOCKER_USERNAME` matches your Docker Hub namespace exactly
  - `DOCKER_PASSWORD` is an Access Token with **Read & Write**
  - the repository exists on Docker Hub (e.g. `chrisrattana/calculator`)
