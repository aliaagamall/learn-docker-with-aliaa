# Lesson 4: Hot Reloading with Docker

## Overview

In this lesson, I will explore hot reloading with Docker. I want to change my code while it’s running in a container and see the changes right away without rebuilding the image every time. This is a common need when I work locally, and I’ll use my Flask app to show how to do it and what challenges I might face.

## The Problem: Changes Don’t Show Without Rebuilding

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

I built an image and ran a container with these commands:

### Step 1: Build the Docker Image

**Command**:

```bash
docker build -t my-flask-app .
```

- **docker build**: Creates a Docker image.
- **-t my-flask-app**: Names the image `my-flask-app`.
- **.** : Uses the current folder as the build context.
- **Output**: I see build steps and then:

```
Successfully built <image-id>
Successfully tagged my-flask-app:latest
```

### Step 2: Run the Container

**Command**:

```bash
docker run --name my-flask-app-container -d -p 5000:5000 my-flask-app
```

- **docker run**: Starts a new container from the image.
- **--name my-flask-app-container**: Sets the container name.
- **-d**: Runs it in the background.
- **-p 5000:5000**: Maps port 5000 on my host to port 5000 in the container.
- **my-flask-app**: Uses the image I built.
- **Output**: I get a container ID (e.g., `bf0e06d53fb8`), and the app runs at `http://localhost:5000`.

Now, I change the code to:

```python
from flask import Flask
import pandas as pd

app = Flask(__name__)

@app.route('/')
def hello():
    return 'Hello, World, My Name Is Aliaa🥰'

if __name__ == '__main__':
    app.run(host="0.0.0.0", port=5000, debug=True)
```

I reload the page at `http://localhost:5000`, but the change doesn’t show! Why? I check the files inside the container.

### Step 3: Enter the Container’s Terminal

**Command**:

```bash
docker exec -it my-flask-app-container bash
```

- **docker exec**: Runs a command in the running container.
- **-it**: Gives me an interactive terminal.
- **my-flask-app-container**: The container name.
- **bash**: Opens a shell inside the container.
- **Output**: I see:

```
root@bf0e06d53fb8:/myflaskapp#
```

### Step 4: List and Check Files

**Command** (inside the container):

```bash
ls
cat app.py
```

- **ls**: Lists files in `/myflaskapp`.
- **cat app.py**: Shows the content of `app.py`.
- **Output**:

  ```
  app.py  requirements.txt
  from flask import Flask
  import pandas as pd
  app = Flask(__name__)
  
  @app.route('/')
  def hello():
      return 'Hello, World'
  
  if __name__ == '__main__':
      app.run(host="0.0.0.0", port=5000, debug=True)
  ```

**Explanation**: The old code is still there because the image was built with the original `app.py`. The container uses the image’s files, not my local changes. To see the new code, I need to rebuild the image and restart the container, which is annoying when I’m testing often👎.

## The Solution: Synchronizing Local and Container Files

I can sync my local folder with the container using a **volume**. This lets my local changes appear in the container without rebuilding.

### Step 5: Run Container with Volume

**Command**:

```bash
docker run --name my-flask-app-container -v C:/test:/myflaskapp -d -p 5000:5000 my-flask-app
```

- **docker run**: Starts a new container from the image.
- **--name my-flask-app-container**: Sets the container name.
- **-v C:/test:/myflaskapp**: Mounts my local folder (`C:/test`) to `/myflaskapp` in the container. (Replace `C:/test` with the path to your project folder.)
- **-d**: Runs the container in the background.
- **-p 5000:5000**: Maps port 5000 on the host to port 5000 in the container.
- **my-flask-app**: Uses the image I built.
- **Output**: A new container ID (e.g., `62dbc200be14`).

**Syntax Explanation**:

- **-v** or **--volume**: Links a folder on my computer (host) to a folder in the container. The format is `host-path:container-path`.
  - `C:/test`: The folder on my computer where my project files are.
  - `/myflaskapp`: The folder inside the container where files will go.
- This syncs changes, so if I edit `app.py` locally, the container sees it right away.

Now, I change `app.py` to:

```python
from flask import Flask
import pandas as pd

app = Flask(__name__)

@app.route('/')
def hello():
    return 'Hello, World, My Name Is Aliaa🥰'

if __name__ == '__main__':
    app.run(host="0.0.0.0", port=5000, debug=True)
```

I reload `http://localhost:5000`, and the new text appears! The volume syncs my local changes.

### Step 6: Check Files in the New Container

**Command**:

```bash
docker exec -it my-flask-app-container bash
ls
cat app.py
```

- **Output**:

  ```
  Dockerfile  __pycache__  app.py  requirements.txt  t.ipynb
  from flask import Flask
  import pandas as pd
  app = Flask(__name__)
  
  @app.route('/')
  def hello():
      return 'Hello, World, My Name Is Aliaa🥰'
  
  if __name__ == '__main__':
      app.run(host="0.0.0.0", port=5000, debug=True)
  ```

**Explanation**: The new `app.py` shows my change, but I also see `Dockerfile`, `__pycache__`, `t.ipynb`, and other files I ignored before. Why? The volume mounts my entire local folder, overriding the image.

## The Challenge: Changes in Container Affect Local Folder

There’s another issue. If I make changes inside the container, they sync back to my local folder because of the volume.

### Example

I enter the container:

```bash
docker exec -it my-flask-app-container bash
```

I create a new file inside:

```bash
touch newfile.txt
ls
```

- **Output**:

  ```
  Dockerfile  __pycache__  app.py  requirements.txt  t.ipynb  newfile.txt
  ```

If I check my local folder (`C:/test`), I’ll see `newfile.txt` there too😫! This happens because the `-v` option creates a two-way sync. Any change in the container affects my local folder, which I might not want.

## Next Steps

In the next lesson, I will figure out how to control this. I might use `.dockerignore` better or add rules to limit what syncs with the volume.

## Summary

Rebuilding is needed for code changes, but a volume with `-v` lets me see local changes instantly. However, it brings back ignored files and syncs container changes to my local folder. I’ll solve this next time!