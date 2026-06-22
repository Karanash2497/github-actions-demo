# GitHub Actions Demo Project

A minimal Python Flask app wired up with two GitHub Actions workflows for hands-on practice.

---

## Project Structure

```
.
├── app/
│   └── main.py                         # Flask app
├── tests/
│   └── test_main.py                    # pytest unit tests
├── .github/
│   └── workflows/
│       ├── ci.yml                      # CI: lint → test → docker build
│       └── ecr-push.yml                # Push Docker image to AWS ECR
├── Dockerfile
├── requirements.txt
└── README.md
```

---

## Workflows

### 1. CI Pipeline (`ci.yml`)

**Triggers:** push to `main`/`develop`, or any PR to `main`

| Job | What it does |
|-----|-------------|
| `lint` | Runs `flake8` on the codebase |
| `test` | Runs `pytest` (needs lint to pass) |
| `build` | Builds the Docker image (needs tests to pass) |

### 2. Build & Push to ECR (`ecr-push.yml`)

**Triggers:** push to `main`, or manual via **Actions → Run workflow**

Authenticates to AWS, logs into ECR, builds the image, and pushes two tags:
- `:latest`
- `:<git-sha>` (for traceability)

Also writes a job summary visible in the GitHub Actions UI.

---

## Setup

### 1. Clone & push to your GitHub repo

```bash
git init
git remote add origin https://github.com/<your-username>/github-actions-demo.git
git add .
git commit -m "Initial commit"
git push -u origin main
```

### 2. Create ECR Repository (one-time)

```bash
aws ecr create-repository \
  --repository-name github-actions-demo \
  --region ap-south-1
```

### 3. Add GitHub Secrets

Go to **Settings → Secrets and variables → Actions → New repository secret**

| Secret Name | Value |
|-------------|-------|
| `AWS_ACCESS_KEY_ID` | Your IAM user access key |
| `AWS_SECRET_ACCESS_KEY` | Your IAM user secret key |

> **Tip:** Create a dedicated IAM user with only `AmazonEC2ContainerRegistryPowerUser` policy attached.

### 4. Trigger the workflows

- **CI:** Push any commit or open a PR → `ci.yml` runs automatically
- **ECR Push:** Merge to `main`, or go to **Actions → Build & Push to ECR → Run workflow**

---

## Run Locally

```bash
pip install -r requirements.txt

# Run app
python -m app.main

# Run tests
pytest tests/ -v

# Build Docker image
docker build -t github-actions-demo .
docker run -p 8080:8080 github-actions-demo
```
