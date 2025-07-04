# Lesson 2: Basic Docker Commands with a Python Example

## Overview

This lesson focuses on the basic Docker commands needed to build, run, and manage containers. I will explain each command, detailing its required and optional components, and then demonstrate their use with a simple Python Flask application. Finally, I will provide a simplified diagram with a trace of the Dockerfile execution to illustrate how the image is built.

## Basic Docker Commands

Below are the core Docker commands used to work with containers, with their required and optional parts clearly defined.

### 2.1 Building the Docker Image

**Command**:

```bash
docker build -t image-name .
```

**Breakdown**:

| Part | Description | Required/Optional |
|------|-------------|-------------------|
| `docker build` | Initiates the image build process using the Dockerfile. | Required |
| `-t image-name` | Assigns the name `image-name` to the image. | Optional (if omitted, the image is unnamed but can be referenced by ID). |
| `.` | Specifies the build context (current directory). | Required |

**Example Output**:

```
Sending build context to Docker daemon  5.632kB
Step 1/7 : FROM python:3.9-slim
 ---> a1b2c3d4e5
...
Successfully tagged image-name:latest
```

### 2.2 Running the Container

**Command**:

```bash
docker run -d -p 5000:5000 --name container-name image-name
```

**Breakdown**:

| Part | Description | Required/Optional |
|------|-------------|-------------------|
| `docker run` | Starts a new container from the specified image. | Required |
| `-d` | Runs the container in detached (background) mode. | Optional (if omitted, the container runs in foreground, showing logs). |
| `-p 5000:5000` | Maps port 5000 on the host to port 5000 in the container. | Optional (required only if external access to the application is needed). |
| `--name container-name` | Assigns the name `container-name` to the container. | Optional (if omitted, Docker assigns a random name). |
| `image-name` | Specifies the image to use. | Required |
---

**Example Output**:

```
1a2b3c4d5e6f7g8h9i0j
```

### 2.3 Inspecting Containers

**Command**:

```bash
docker ps
```

**Breakdown**:

| Part | Description | Required/Optional |
|------|-------------|-------------------|
| `docker ps` | Lists all running containers. | Required |

**Example Output**:

```
CONTAINER ID   IMAGE           PORTS                    NAMES
abc123         image-name    0.0.0.0:5000->5000/tcp   container-name
```

### 2.4 Stopping and Removing the Container

**Command**:

```bash
docker rm container-name -f
```

**Breakdown**:

| Part | Description | Required/Optional |
|------|-------------|-------------------|
| `docker rm` | Removes the specified container. | Required |
| `container-name` | Specifies the container name or ID to remove. | Required |
| `-f` | Forces removal, stopping the container if it is running. | Optional (if omitted, you must stop the container first using `docker stop`). |

### 2.5 Removing the Image

**Command**:

```bash
docker rmi image-name
```

**Breakdown**:

| Part | Description | Required/Optional |
|------|-------------|-------------------|
| `docker rmi` | Removes the specified image. | Required |
| `image-name` | Specifies the image name or ID to remove. | Required |

## Example Application

To demonstrate these commands, we use a simple Flask application.

### 2.6 Flask Application Code

The application code (`app.py`) is as follows:

```python
from flask import Flask
import pandas as pd

app = Flask(__name__)

@app.route('/')
def hello():
    return 'Hello, World'

if __name__ == '__main__':
    app.run(host="0.0.0.0", port=5000, debug=True)
```

This code:
- Creates a Flask web server.
- Displays "Hello, World" at the root URL (`/`).
- Runs on port 5000.

### 2.7 Python Dependencies

The dependencies are listed in `requirements.txt`:

```
flask
pandas
```

This file specifies the Python packages needed by the application.

### 2.8 Dockerfile

The `Dockerfile` defines how to build the Docker image:

```dockerfile
FROM python:3.9-slim
WORKDIR /myflaskapp
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . /myflaskapp
EXPOSE 5000
CMD ["python", "app.py"]
```

## Commands Execution Trace

Below, trace the execution of the Commands
```
                             +-------------------+
                             |     Dockerfile    |
                             +-------------------+
                                       |
                          docker build -t my-flask-app .
                                       ↓
                             +-------------------+
                             |   Docker Image    |
                             | (Packaged app)    |
                             +-------------------+
                                       |
        docker run -d -p 5000:5000 --name my-flask-app-container my-flask-app
                                       ↓
                             +-------------------+
                             | Docker Container  |
                             | Runs Flask App    |
                             +-------------------+
                                       |
                       Test it: http://localhost:5000

```

### Explanation of Each Command

#### 1. `docker build -t my-flask-app .`

**Purpose:**
Build a Docker image from  Dockerfile.

**Parts:**

* `docker build`: the command to build an image.
* `-t my-flask-app`: the `-t` option tags (names) the image
* `.` : means "use the current directory". 

---

#### 2. `docker run --name my-flask-app-container -d -p 5000:5000 my-flask-app`

**Purpose:**
Run a container from  image.

**Parts:**

* `docker run`: the command to start a container.
* `--name my-flask-app-container`: the container a name.
* `-d`: run in detached mode (in background).
* `-p 5000:5000`: map port 5000 on your machine to port 5000 in the container.
* `my-flask-app`: the image name .


##### Notes on Port Mapping with `-p`
- The `-p 5000:5000` option in the `docker run` command maps a port on the host machine to a port inside the container.
  - The **first number (5000)** represents the port on the host machine. This port can be changed (e.g., to 5001, 5002, etc.) as long as it is available and not used by another application on the host.
  - The **second number (5000)** represents the port inside the container. This port depends on the application’s configuration within the container (e.g., your Flask app is set to use port 5000 with `app.run(host="0.0.0.0", port=5000, debug=True)`), and it typically remains the same unless the application code is modified.
- **Example**:
  - `docker run -d -p 5000:5000 --name flask-container my-flask-app`: Access the app at `http://localhost:5000`.
  - `docker run -d -p 5001:5000 --name flask-container-2 my-flask-app`: Access the app at `http://localhost:5001`.

---

### Dockerfile Execution Trace

```
+-------------------------------+
| Container                     |
|   +-----------------------+   |
|   | myflaskapp            |   | <- WORKDIR /myflaskapp
|   |   - requirements.txt  |   | <- COPY requirements.txt .
|   |   - app.py            |   |    RUN pip install --no-cache-dir -r requirements.txt
|   +-----------------------+   |
|   Trace: Copies app files     |
|   and installs dependencies   |
+-------------------------------+
| Python 3.9 Slim Base Image    |
|   Trace: Provides base env    | <- FROM python:3.9-slim
+-------------------------------+

```

**Trace Explanation**:

1. **Python 3.9 Slim Base Image**: The foundation of the container, providing a minimal environment with Python 3.9 pre-installed.
2. **myflaskapp**: Contains the application files and dependencies.
   - **requirements.txt**: Specifies `flask` and `pandas` for installation.
   - **app.py**: The Flask application code.
   - **Trace**: The `COPY` commands copy these files into the container, and `RUN pip install` installs the dependencies. The `CMD` runs `app.py` to start the server on port 5000.

## Testing the Application

After running the container with:

```bash
docker run -d -p 5000:5000 --name flask-container my-flask-app
```

Navigate to `http://localhost:5000` in a browser. You should see:

```
Hello, World
```

## Commands Summary

| Command                                              | Purpose                 |
| ---------------------------------------------------- | ----------------------- |
| `docker build -t my-flask-app .`                     | Build an image          |
| `docker run --name my-flask-app-container -d -p ...` | Run the container       |
| `docker ps`                                          | List running containers |
| `docker rm ... -f`                                   | Remove a container      |
| `docker rmi ...`                                     | Remove an image         |
| `docker image ls`                                    | List all images         |

---

## Understanding Docker Images and Creating Multiple Containers

The image (Image) in Docker is similar to a class (Class) in Object-Oriented Programming (OOP). Just as you can create multiple objects (instances) from a class, you can create multiple containers (Containers) from a single image. The image serves as a fixed template containing the code and dependencies, allowing you to run several containers from it simultaneously or at different times.

### Notes:
- **Ports**: If you want to access each container via a browser, use different ports on the host machine (e.g., 5000, 5001, 5002) to avoid conflicts.
- **Names**: Ensure container names are unique (e.g., `container-1`, `container-2`), or let Docker assign random names if you omit the `--name` option.
