---
title: Automating Docker Deployment with Github Actions
date: 2025-01-18
---

As developers, we often need to deploy applications to server environments.

In this guide, I'll show you how to automate Docker container deployments using [GitHub Actions](https://github.com/features/actions), [Docker Hub](https://hub.docker.com), and a simple cron job. Whether you're new to Docker or looking to streamline your deployment process, this tutorial will walk you through both **manual** and **automated** approaches.

## Part 1: The Manual Process

Before building an Docker image, we setup a simple web server in `main.py` with two endpoints using [FastAPI](https://fastapi.tiangolo.com).

This is just for demo purposes you can use whatever you like.

### 1. Creating a Sample FastAPI Application 

First, let's create a minimal web server using [FastAPI](https://fastapi.tiangolo.com). 

Create a `main.py` file:

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
def root():
    return "Hello World!!"


@app.get("/{name}")
def get_name(name: str):
    return f"Hello {name.title()}"
```

You can test this locally by running:

```sh
# Install dependencies
pip install fastapi uvicorn

# Run server
uvicorn main:app
```

### 2. Containerizing the Application

We now want containerize the application. Create `Dockerfile` that packages the application:

```Dockerfile
# Use a Python image
FROM python:3.12 

# Install the project into `/app`
WORKDIR /app

# Install dependencies
RUN pip install fastapi uvicorn 

# Copy the application code
COPY . /app

# Expose the port and start the server
EXPOSE 8000
CMD ["uvicorn", "main:app", "--host", "0.0.0.0"]
```

### 3. Building and Publishing the Container

To build the Docker image and get the container into production you need to:

```sh
# Build the image
docker build -t my-username/my-image .

# Run and test the container locally
docker run --name container-name -p 8000:8000 my-username/my-image

# Push to Docker Hub
docker push my-username/my-image
```

### 4. Setting Up Docker on Your Server

Here are the commands you'll need to setup Docker on a Debian-based server. Straight from the Docker [documentation](https://docs.docker.com/engine/install/).

```sh
# Remove old versions
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done

# Install prerequisites
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings

# Add Docker's PGP key and repository
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Setup non-root access
sudo groupadd docker
sudo usermod -aG docker $USER
```

> Remember to log out and back in for the group changes to take effect.

```sh
# Pull the image
docker pull my-username/my-image

# Run the container
docker run --name container-name -p 8000:8000 my-username/my-image
```

## Part 2: Automating the Process

Now let's automate everything we did manually. We'll use GitHub Actions for building and pushing images, and a custom script for automated deployments.

### 1. Setting Up GitHub Actions

Create `.github/workflows/deploy.yml`:

```yml
name: Build and Push Docker Image

on:
  push:
    branches:
      - main

jobs:
  build-and-push:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout
      uses: actions/checkout@v4

    - name: Log in to Docker Hub
      uses: docker/login-action@v3
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3

    - name: Build and Push Docker Image
      uses: docker/build-push-action@v6
      with:
        context: .
        platforms: linux/arm64
        push: true
        tags: my-username/my-image:latest
```

### 2. Automated Deployment Script 

Create a `deploy.sh` script on your server:

```sh
#!/bin/bash
IMAGE="my-username/my-image:latest"
CONTAINER_NAME="my-container-name"

# Pull the latest image
echo "Pulling the latest image..."
docker pull $IMAGE

# Check if the container exists
CONTAINER_EXISTS=$(docker ps -a --filter "name=^/${CONTAINER_NAME}$" --format '{{.Names}}')

# Check if the container is running
CONTAINER_RUNNING=$(docker ps --filter "name=^/${CONTAINER_NAME}$" --filter "status=running" --format '{{.Names}}')

# Get the hash of the currently running image (if the container exists)
if [ -n "$CONTAINER_EXISTS" ]; then
  CURRENT_IMAGE=$(docker inspect --format='{{.Image}}' $CONTAINER_NAME)
else
  CURRENT_IMAGE=""
fi

# Get the hash of the latest image
LATEST_IMAGE=$(docker inspect --format='{{.Id}}' $IMAGE)

if [ "$CONTAINER_RUNNING" ] && [ "$CURRENT_IMAGE" == "$LATEST_IMAGE" ]; then
  echo "The container is already running the latest image. No action needed."
else
  echo "Updating or starting the container..."
  
  if [ -n "$CONTAINER_EXISTS" ]; then
    docker stop $CONTAINER_NAME || true
    docker rm $CONTAINER_NAME || true
  fi
  
  docker run --name $CONTAINER_NAME -d -p 8000:8000 $IMAGE
  echo "Deployment completed."
fi
```

### 3. Setting Up the Cron Job

To automatically check for image updates:

```sh
# Make the deploy script executable
chmod +x deploy.sh

# Add to crontab (runs every 15 minutes)
(crontab -l 2>/dev/null; echo "*/15 * * * * /path/to/deploy.sh >> /var/log/deploy.log 2>&1") | crontab -
```

## Considerations

1. Always use specific image tags in production rather than `latest`
2. Implement health checks in your container
3. Use Docker secrets for sensitive information
4. Implement proper logging and monitoring
5. Consider using Docker Compose for more complex applications

