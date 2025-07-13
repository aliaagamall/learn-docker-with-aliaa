# Lesson 5: One-Way Binding and Data Persistence with Docker

## Overview

In this lesson, I will fix the problem from last time where changes in the container affected my local folder. I’ll use one-way binding to stop this and learn how to save data even when the container stops. I’ll keep using my Flask app to show these ideas.

## The Problem: Two-Way Binding Issues

Last time, I used a volume to sync my local folder with the container, but it was a two-way binding. This meant changes in the container (like adding a file) also changed my local folder, which I didn’t want.

I’m using this Flask app code (`app.py`):

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

And this was my Dockerfile:

```dockerfile
FROM python:3.9-slim

WORKDIR /myflaskapp

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . /myflaskapp

EXPOSE 5000

CMD ["python", "app.py"]
```

I ran the container with:

### Step 1: Run Container with Two-Way Binding

**Command**:

```bash
docker run --name my-flask-app-container -v C:/test:/myflaskapp -d -p 5000:5000 my-flask-app
```

### Step 2: Enter the Container’s Terminal

**Command**:

```bash
docker exec -it my-flask-app-container bash
```

- **Output**:

```
root@0b4fe3c31f02:/myflaskapp#
```

### Step 3: List and Try to Create a File

**Command** (inside the container):

```bash
ls
touch t.text
```

- **ls**: Lists files in `/myflaskapp`.

- **touch t.text**: Tries to create a new file.

- **Output**:

  ```
  Dockerfile  __pycache__  app.py  requirements.txt  t.ipynb
  touch: cannot touch 't.text': Read-only file system
  ```

**Explanation**: The folder is writable because of two-way binding. If I could create `t.text`, it would appear in `C:/test` too.

## The Solution: One-Way Binding with :ro

I want one-way binding so my local changes go to the container, but container changes don’t affect my local folder. I’ll use `:ro` (read-only) to do this.

### Step 4: Run Container with One-Way Binding

**Command**:

```bash
docker run --name my-flask-app-container -v C:/test:/myflaskapp:ro -d -p 5000:5000 my-flask-app
```

- **docker run**: Starts a new container.
- **--name my-flask-app-container**: Sets the container name.
- **-v C:/test:/myflaskapp:ro**: Mounts `C:/test` to `/myflaskapp` as read-only (one-way sync from host to container).
- **-d**: Runs in the background.
- **-p 5000:5000**: Maps the ports.
- **my-flask-app**: Uses the image.
- **Output**: A container ID (e.g., `0b4fe3c31f02`).

**Syntax Explanation**:

- **-v host-path:container-path:ro**: The `:ro` makes the container’s folder read-only. My local folder (`C:/test`) sends files to `/myflaskapp`, but the container can’t change them.

### Step 5: Enter and Test the Container

**Command**:

```bash
docker exec -it my-flask-app-container bash
ls
touch t.text
```

- **Output**:

  ```
  Dockerfile  __pycache__  app.py  requirements.txt  t.ipynb
  touch: cannot touch 't.text': Read-only file system
  ```

**Explanation**: I can’t create `t.text` because `/myflaskapp` is read-only. My local folder stays safe!

### Alternative Syntax with ${PWD}

I can also use `${PWD}` instead of the full path:

**Command**:

```bash
docker run --name my-flask-app-container -v ${PWD}:/myflaskapp:ro -d -p 5000:5000 my-flask-app
```

- **${PWD}**: Gets the current folder path automatically (e.g., my project folder).
- **Output**: Same as above, a container ID.

**Explanation**: This is easier than typing the full path like `C:/test`.

## Do I Need to Bind Every Folder?

No, I don’t need to bind every folder. I only need to bind the source files where I make changes, like `app.py`. For example, I don’t want `requirements.txt` to be deleted in the container if I delete it locally.

### Better Solution: Bind Specific Folder

**Command**:

```bash
docker run --name my-flask-app-container -v ${PWD}/src:/myflaskapp/src:ro -d -p 5000:5000 my-flask-app
```

- **-v ${PWD}/src:/myflaskapp/src:ro**: Mounts only the `src` folder from my project to `/myflaskapp/src` as read-only.
- **Output**: A container ID.

**Explanation**: Changes outside `src` (e.g., `requirements.txt`) won’t affect the container. Only files in `src` (like `app.py`) sync.

### Step 6: Enter and Check the Container

**Command**:

```bash
docker exec -it my-flask-app-container bash
ls
```

- **Output**:

  ```
  requirements.txt  src
  ```

**Explanation**: Only `requirements.txt` and `src` are visible because the volume mounts only the `src` folder.

## Update the Dockerfile

Since I’m using a `src` folder, I need to update my Dockerfile:

```dockerfile
FROM python:3.9-slim

WORKDIR /myflaskapp

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . /myflaskapp

EXPOSE 5000

CMD ["python", "src/app.py"]  <-- Changed to run from src folder
```

### Step 7: Rebuild the Image

**Command**:

```bash
docker build -t my-flask-app .
```

- **Output**:

```
Successfully built <new-image-id>
Successfully tagged my-flask-app:latest
```

### Step 8: Run the Updated Container

**Command**:

```bash
docker run --name my-flask-app-container -v ${PWD}/src:/myflaskapp/src:ro -d -p 5000:5000 my-flask-app
```

- **Output**: A new container ID.

**Explanation**: The app runs from `/myflaskapp/src/app.py` now.

## Issue with Ignored Files

Last time, ignored files (like `Dockerfile`, `__pycache__`, etc.) reappeared because the volume mounted my whole folder. Now, by binding only the `src` folder, those files are ignored again. This happens because `.dockerignore` applies, and since it’s outside `src`, it prevents those files from being included in the mounted folder.

- **Check**:

```bash
docker exec -it my-flask-app-container bash
ls
```

- **Output**:

  ```
  requirements.txt  src
  ```

**Explanation**: The ignored files are gone because the volume only mounts `src`, and `.dockerignore` still works for the image build.

## Data Persistence with Volumes

A container is a process running in memory. If I add a database and want to save data , it will disappear when the container stops. Volumes help by storing data on my hard drive.

**Example**: I’ll explore this in the another lesson with a database.

## Summary

I learned to use `:ro` for one-way binding to protect my local folder. I can bind specific folders like `src` and update the Dockerfile. The ignored files issue is fixed by selective mounting.