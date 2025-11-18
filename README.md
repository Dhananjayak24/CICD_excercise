# CI/CD Pipeline: GitHub Actions → AWS ECR → Docker

- This guide walks through setting up a CI/CD pipeline that:

- Builds a Docker image from a project

- Pushes the image to AWS ECR

- Uses GitHub Actions for automation

- Uses Docker Compose locally

  ## Prerequisites

- Docker Desktop installed

- Git installed

- AWS account

- GitHub account

  ## 1. Setup Project Repo

1. Download the practice project from Coursera
2. Move it into your working directory
3. Initialize Git:
```
git init
```
4. Verify user name
```
git config user.name
```
5. Create a new GitHub repository (name + description)
6. Add remote origin:
```
git remote add origin git@github.com:Dhananjayak24/CICD_excercise.git
```
7. Create main branch (If required create dev branch too
```
git checkout -b main
git checkout -b dev
```
## 2. Run Project Locally with Docker

1. Start Docker Desktop
2. Run docker compose:
```
   docker compose up
```
## 3. Create an AWS ECR Repository

1. Go to AWS Console → Search ECR
2. Create repository
3. Repository name: **coursera/vue-docker**
4. Keep default settings

## 4. Create AWS IAM User for CI/CD
1. Search IAM → Users → Create user
2. User name: **GitHub-vue-user**
3. Attach policy: **AmazonEC2ContainerRegistryFullAccess**
4. Create user
5. Open the user → Security credentials tab
6. Create Access Key → Command Line Interface
7. Copy:
    - Access Key
    - Secret Key
  
## 5. Add Secrets in GitHub Repo

GitHub Repo → Settings → Secrets & variables → Actions → New repository secret

| Name                    | Value                     |
| ----------------------- | ------------------------- |
| `AWS_ACCESS_KEY_ID`     | From IAM                  |
| `AWS_SECRET_ACCESS_KEY` | From IAM                  |
| `ECR_REPO`              | URI from AWS ECR repo     |
| `AWS_REGION`            | Example: `ap-southeast-2` |

*Note*: `AWS_REGION` does not need to be a secret, but adding it is OK here.

## 6. Add GitHub Actions CI Pipeline

Create the file:
```
.github/workflows/ci.yml
```
Paste this

```
name: CI push to ECR pipeline

on:
  push:
    branches:
      - main
  workflow_dispatch:

jobs:
  docker-build-push:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ secrets.AWS_REGION }}

      - name: Amazon ECR Login
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2

      - name: Build docker image
        run: docker build -t ${{ secrets.ECR_REPO }}:latest .

      - name: Push docker image to Amazon ECR
        run: docker push ${{ secrets.ECR_REPO }}:latest
```

## 7. Push Code to GitHub
```
git add .
git commit -m "chore: Initial commit"
git switch main
git merge dev
git push origin main
```
This will automatically start the CI pipeline under the Actions tab.

## 8. Retrigger Actions (if needed)

If secrets were updated and no file changed:

```
git commit --allow-empty -m "Trigger workflow"
git push origin main
```

  
