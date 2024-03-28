### Streamlining Full Stack Development with Docker, Docker-Compose, Docker Hub, and GitHub Actions

## Introduction

In today's fast-paced software development landscape, Continuous Integration and Continuous Deployment (CI/CD) 
practices are essential for maintaining high-quality code and delivering features to users quickly and efficiently. 
Docker, Docker-Compose, Docker Hub, and GitHub Actions are powerful tools that streamline the development process and automate 
CI/CD workflows.

### Section 1: Docker Basics

Docker is a popular platform for developing, shipping, and running applications in containers. Containers encapsulate an application and its dependencies, allowing it to run consistently across different environments. Key concepts include:

1. Dockerfile: A text file that contains instructions for building a Docker image.
2. Docker Image: A lightweight, standalone, and executable package that contains all the necessary dependencies to run an application.
3. Docker Container: An instance of a Docker image that runs as a process on the host machine.
4. Docker-Compose: It is a tool for defining and running multi-container applications. It is the key to unlocking a streamlined and efficient development and deployment experience.

**Benefits of using Docker include:**
 
 1. Consistency: Ensures that applications run the same way in development, testing, and production environments.
 2. Isolation: Each container runs independently, preventing conflicts between dependencies.
 3. Portability: Docker images can be easily shared and deployed across different platforms.

### Section 2: Docker Hub:

Docker Hub is a cloud-based repository for storing, sharing, and distributing Docker images. Key features include:

1. Public and Private Repositories: Docker Hub allows users to create both public and private repositories to manage Docker images.
2. Versioning: Docker Hub supports versioning for images, making it easy to track changes and roll back to previous versions.
3. Automated Builds: Docker Hub can automatically build Docker images from source code repositories such as GitHub, GitLab, and Bitbucket.

Creating a Docker Hub account and repository is straightforward, allowing developers to share their Docker images with team members and the wider community.

### Section 3: GitHub Actions for CI/CD:

GitHub Actions is a powerful CI/CD platform built into GitHub, enabling developers to automate workflows directly from their repositories. Key features include:

1. YAML-based Workflows: GitHub Actions workflows are defined using YAML syntax, making them easy to read and maintain.
2. Trigger Events: Workflows can be triggered by various events, such as push, pull request, or schedule.
3. Marketplace Integrations: GitHub Actions provides a marketplace of pre-built actions that can be used to automate common tasks.

### Section 4: Setting up CI/CD Pipeline with GitHub Actions:

Setting up a CI/CD pipeline with GitHub Actions involves defining workflows that automate tasks such as building, testing, and deploying applications. A typical CI/CD pipeline may include the following steps:

1. Build: Compile the application code and create a Docker image.
2. Test: Run unit tests, integration tests, and other automated tests to ensure code quality.
3. Deploy: Deploy the Docker image to a target environment, such as a staging or production server.

GitHub Actions integrates seamlessly with Docker Hub, allowing developers to push Docker images to repositories as part of their CI/CD workflows.

### Section 5: Best Practices and Tips:

To ensure the success of CI/CD pipelines using Docker, Docker-Compose, Docker Hub, and GitHub Actions, it's essential to follow best practices:

1. Optimize Dockerfile: Keep Dockerfiles concise and minimize the number of layers to improve build performance.
2. Security Considerations: Regularly scan Docker images for vulnerabilities and apply security patches promptly.
3. Pipeline Optimization: Continuously monitor and optimize CI/CD pipelines for speed, reliability, and efficiency.


### Section 6: Real-World Use Cases:

***Project Structure:***

## Frontend React Application:
```
frontend/
├── public/
│   ├── index.html
├── src/
│   ├── components/
│   │   ├── App.js
│   │   └── ...
│   ├── index.js
│   └── ...
├── Dockerfile
├── .dockerignore
├── package.json
├── yarn.lock
├── entrypoint.sh
└── ...
```

- **Dockerfile:** Contains instructions to build the Docker image for the frontend React application.
- **.dockerignore:** Specifies files and directories to be excluded from the Docker build context.

### Dockerfile
```
# Builder Stage
FROM node:21.6.0 as builder

WORKDIR /app

# Copy only the package files first to leverage Docker layer caching
COPY package.json yarn.lock ./

# Update package lists and upgrade packages
RUN apt-get update && \
    apt-get upgrade -y && \
    apt-get install -y curl

# Install dependencies without husky hooks
RUN yarn install --ignore-scripts

# Copy the entire application
COPY . .

# Build the application
RUN yarn build

# Production Stage
FROM nginx:stable-perl

WORKDIR /usr/share/nginx/html

# Create the necessary directory and set permissions
RUN mkdir -p /var/cache/nginx/client_temp && \
    chown -R nginx:nginx /var/cache/nginx

RUN mkdir -p /var/run/nginx && \
    chown -R nginx:nginx /var/run/nginx

# Remove default Nginx contents
RUN rm -rf ./*

# Copy the built application from the builder stage
COPY --from=builder /app/dist .

# Copy the entrypoint script
COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

# Set the entrypoint
ENTRYPOINT ["/entrypoint.sh"]

USER root

# Start Nginx
CMD ["nginx", "-g", "daemon off;"]
```

*** Github Actions Workflow *** 
Add a workflow file in your frontend project e.g (*.github/workflow/build-and-push.yml*)
```
name: Build and Push Docker Image

on:
  push:
    branches:
      - master
      - feature/*

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Determine Docker Tag
        id: tag
        run: |
          if [[ $GITHUB_REF == 'refs/heads/master' ]]; then
            echo 'DOCKER_TAG=latest' >> $GITHUB_ENV
          elif [[ $GITHUB_REF == 'refs/heads/feature/'* ]]; then
            echo "DOCKER_TAG=feature-${GITHUB_REF#refs/heads/feature/}" >> $GITHUB_ENV
          else
            echo "DOCKER_TAG=unknown" >> $GITHUB_ENV
          fi

      - name: Print Docker tag
        run: |
          echo "Docker Tag: ${{ env.DOCKER_TAG }}"

      - name: Build Docker image
        run: docker build -t docker-hub-repo/frontend:${{ env.DOCKER_TAG }} .

      - name: Log in to Docker Hub
        run: echo "${{ secrets.DOCKERHUB_TOKEN }}" | docker login -u "${{ secrets.DOCKERHUB_USERNAME }}" --password-stdin

      - name: Push Docker image to Docker Hub
        run: docker push docker-hub-repo/frontend:${{ env.DOCKER_TAG }}


## Backend Django Application:

```
backend/
├── myapp/
│   ├── models.py
│   ├── views.py
│   └── ...
├── manage.py
├── Dockerfile
├── .dockerignore
├── requirements.txt
├── entrypoint.sh
└── ...
```

- **Dockerfile:** Contains instructions to build the Docker image for the backend Django application.
- **.dockerignore:** Specifies files and directories to be excluded from the Docker build context.

### Dockerfile
```
# === Stage 1: Python Build Stage ===
FROM python:3.9.18-slim-bullseye AS python-build-stage

# Install Git LFS
RUN apt-get update \
    && apt-get install -y git-lfs \
    && git lfs install \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Set build environment to 'local' by default
ARG BUILD_ENVIRONMENT=local

# Set the working directory for the build stage
WORKDIR /build

# Copy only the requirements file to the build stage
COPY ./requirements.txt .

# Install required system dependencies for building Python packages
RUN apt-get update \
    && apt-get install --no-install-recommends -y \
        build-essential \
        libpq-dev \
        gettext \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Create Python Dependency and Sub-Dependency Wheels.
RUN pip install --no-cache-dir --upgrade pip \
    && pip wheel --no-cache-dir --no-deps --wheel-dir /wheels -r requirements.txt


# === Stage 2: Python Run Stage ===
FROM python:3.9.18-slim-bullseye AS python-run-stage

# Set build environment to 'local' by default
ARG BUILD_ENVIRONMENT=local

# Set the working directory for the run stage
WORKDIR /app

# Copy the Python wheels from the build stage to the run stage
COPY --from=python-build-stage /wheels /wheels

# Install apt packages for the run stage
RUN apt-get update \
    && apt-get install --no-install-recommends -y \
        sudo \
        git \
        bash-completion \
        nano \
        ssh \
        curl \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Create a non-root user and add it to sudoers
RUN groupadd --gid 1000 dev-user \
    && useradd --uid 1000 --gid dev-user --shell /bin/bash --create-home dev-user \
    && echo "dev-user ALL=(root) NOPASSWD:ALL" > /etc/sudoers.d/dev-user \
    && chmod 0440 /etc/sudoers.d/dev-user

# Install required system dependencies for the run stage
RUN apt-get update \
    && apt-get install --no-install-recommends -y \
        libpq-dev \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* \
    && pip install --no-cache-dir --upgrade pip

COPY ./requirements.txt .

# Install Python dependencies from wheels
RUN pip install --no-cache-dir --find-links=/wheels -r requirements.txt \
    && rm -rf /root/.cache \
    && rm -rf /wheels

# Copy entrypoint and start scripts
COPY ./entrypoint.sh /entrypoint.sh

# Copy application code to the working directory
COPY . .

# Set the entrypoint
ENTRYPOINT ["/entrypoint.sh"]

```

*** Github Actions Workflow *** 
Add a workflow file in your backend project e.g (*.github/workflow/build-and-push.yml*)

```
name: Build and Push Docker Image

on:
  push:
    branches:
      - master
      - feature/*

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Determine Docker Tag
        id: tag
        run: |
          if [[ $GITHUB_REF == 'refs/heads/master' ]]; then
            echo 'DOCKER_TAG=latest' >> $GITHUB_ENV
          elif [[ $GITHUB_REF == 'refs/heads/feature/'* ]]; then
            echo "DOCKER_TAG=feature-${GITHUB_REF#refs/heads/feature/}" >> $GITHUB_ENV
          else
            echo "DOCKER_TAG=unknown" >> $GITHUB_ENV
          fi

      - name: Print Docker tag
        run: |
          echo "Docker Tag: ${{ env.DOCKER_TAG }}"

      - name: Log in to Docker Hub
        run: echo "${{ secrets.DOCKERHUB_TOKEN }}" | docker login -u "${{ secrets.DOCKERHUB_USERNAME }}" --password-stdin


      - name: Build and Push Docker image
        uses: docker/build-push-action@v2
        with:
          context: .
          file: ./Dockerfile
          tags: docker-hub-repo/backend:${{ env.DOCKER_TAG }}
          push: true
        env:
          DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
          DOCKERHUB_TOKEN: ${{ secrets.DOCKERHUB_TOKEN }}

```


