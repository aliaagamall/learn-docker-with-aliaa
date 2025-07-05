# Lesson 3: Docker Optimization with .dockerignore

## Overview

In this lesson, we will learn how to optimize our Docker build process using a `.dockerignore` file. I will use the same Flask application example and improve how we copy files into the container. This will make our Docker images smaller and faster to build by ignoring unnecessary files.

## The Problem: Unnecessary Files in the Container

We are using this Flask app code (`app.py`):

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

And this Dockerfile:

```dockerfile
FROM python:3.9-slim

WORKDIR /myflaskapp

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . /myflaskapp

EXPOSE 5000

CMD ["python", "app.py"]
```

When we use `COPY . /myflaskapp`, it copies all files from our project folder into the container. But do we need all of them? For example, files like `Dockerfile` or `__pycache__` are not needed inside the container. This is similar to the `.gitignore` file in Git, which helps us ignore files we don’t want to track.

Let’s check what’s inside the container to understand this better.

## Step-by-Step Guide

### Step 1: Build the Docker Image

**Command**:

```bash
docker build -t my-flask-app .
```

- **docker build**: Starts the process to create a Docker image.
- **-t my-flask-app**: Gives the image a name (`my-flask-app`).
- **.** : Uses the current folder as the build context (where the Dockerfile is). 
- **Output**: You will see build steps in the terminal, ending with:

```
Successfully built <image-id>
Successfully tagged my-flask-app:latest
```

**Next**: The image is ready, but we haven’t run it yet.

### Step 2: Run the Container

**Command**:

```bash
docker run --name my-flask-app-container -d -p 5000:5000 my-flask-app
```

- **docker run**: Starts a new container from the image.
- **--name my-flask-app-container**: Sets the container name.
- **-d**: Runs the container in the background.
- **-p 5000:5000**: Maps port 5000 on the host to port 5000 in the container.
- **my-flask-app**: Uses the image we built. 
- **Output**: A container ID (e.g., `d157ea0af12c`) appears, showing the container is running. **Next**: We will check the container’s files.

### Step 3: Enter the Container’s Terminal

**Command**:

```bash
docker exec -it my-flask-app-container bash
```

- **docker exec**: Runs a command in a running container.
- **-it**: Allows interactive terminal access.
- **my-flask-app-container**: The name of the container.
- **bash**: Opens a bash shell inside the container. 
- **Output**: You will see a prompt like:

```
root@d157ea0af12c:/myflaskapp#
```

**Next**: List the files to see what’s inside.

### Step 4: List Files in the Container

**Command** (inside the container):

```bash
ls
```

- **ls**: Lists all files and folders in the current directory (`/myflaskapp`). 
- **Output**: You might see something like:

```
Dockerfile  __pycache__  app.py  requirements.txt  t.ipynb
```

**Explanation**: This shows all files copied into the container. Files like `Dockerfile`, `__pycache__`, and `t.ipynb` are not needed for the app to run. They just make the image bigger and slower to build.

## Solution: Using .dockerignore

We don’t need all these files in the container. Let’s create a `.dockerignore` file to ignore them, similar to `.gitignore` in Git.

### Step 5: Create the .dockerignore File

- Create a file named `.dockerignore` in the same folder as your Dockerfile.
- Add the following lines to ignore unnecessary files:

```
Dockerfile
__pycache__/
*.pyc
*.pyo
*.ipynb
*.log
.env
*.db
*.sqlite3
.git
.gitignore
.vscode/
.idea/
```

- **What this does**: Tells Docker not to copy these files or folders during the `COPY . /myflaskapp` step.
- **Add your file**: If there’s another file you don’t want (e.g., `notes.txt`), add it to the list (e.g., `notes.txt`).

**Output**: No immediate output, but the `.dockerignore` file is now part of your project.

## Step 6: Rebuild the Image

**Command**:

```bash
docker build -t my-flask-app .
```

- **docker build**: Rebuilds the image with the new `.dockerignore` file.

```
Successfully built <new-image-id>
Successfully tagged my-flask-app:latest
```

**Next**: The new image ignores the listed files.

### Step 7: Run a New Container

**Command**:

```bash
docker run --name my-flask-app-container-new -d -p 5001:5000 my-flask-app
```

-  **Next**: Check the new container’s files.

### Step 8: Enter the New Container’s Terminal

**Command**:

```bash
docker exec -it my-flask-app-container-new bash
```

- **docker exec**: Runs a command in the new container.

```
root@151dafce288e:/myflaskapp#
```

**Next**: List the files again.

### Step 9: List Files in the New Container

**Command** (inside the container):

```bash
ls
```

- **ls**: Lists files in `/myflaskapp`. **Output**: You might see:

```
app.py  requirements.txt
```

**Explanation**: The `Dockerfile`, `__pycache__`, `t.ipynb`, and other ignored files are gone, leaving only the essential files (`app.py` and `requirements.txt`).

## Summary

In this lesson, we learned:

- The `COPY .` command copies all files, but not all are needed.
- A `.dockerignore` file helps ignore unnecessary files like `Dockerfile` and `__pycache__`.
- Rebuilding the image with `.dockerignore` makes the container lighter and faster.