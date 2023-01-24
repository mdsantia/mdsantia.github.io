---
layout: single
title: "Production Dockerfile Tips and Tricks"
date: 2023-01-24
mathjax: true
---

Container images are the backbone of a lot of modern workflows, especially in the age of Kubernetes and friends. Building images
that are small, secure, and efficient is crucial for production level applications. In this post we will cover some quick tips and 
tricks for writing better Dockerfiles (you don't actually have to use Docker but the name Dockerfile is probably gonna stick around).

The TLDR version of this post is:
* Use the smallest base image possible
* Make builds deterministic
* Run with a privileged user (never as root)
* Separate dependencies from source code
* Keep things that change more frequently closer to the bottom of the Dockerfile
* The smaller the final image, the better

Instead starting with a bad Dockerfile and building up a better one, lets just start with a production level Dockerfile and use that
as our example to explain some concepts.

```dockerfile
FROM python:3.10.9-slim-bullseye

ENV USER=rta

WORKDIR /app

RUN useradd -mrU $USER && \
    chown -R $USER:$USER /app

COPY src/requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt && \
    rm -f requirements.txt

COPY --chown=$USER:$USER src/main.py /app

USER $USER
ENTRYPOINT ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8080"]
```

### Small base image and deterministic builds
```dockerfile
FROM python:3.10.9-slim-bullseye
```
Ok, lets start at the top, notice we use a "slim" python image and we also specify the exact python version. Specifying the exact python version helps with making our build deterministic. And choosing the slim image helps keep the final image smaller. In general there are 3ish main flavors of python images:
```bash
python 3.10-alpine         8a3a8409a638   Less than a second ago   50.1MB
python 3.10-slim-bullseye  72fb1d206063   Less than a second ago   126MB
python 3.10                109421b4b8cd   Less than a second ago   921MB
```
Although the Alpine image is the smallest, it can be hard to work with sometimes. I would recommend always starting with the slim image and going from there. It is very unlikely that
you will need a full debian or ubuntu OS in your Docker image.

So we have a small (mostly deterministic) base image. If we really want this truly deterministic then we could use the full image digest (`docker images --digests | grep python`) instead of the tag but this may be overkill and can make your Dockerfile less readable. 

### Privileged non-root user
```dockerfile
ENV USER=rta

WORKDIR /app

RUN useradd -mrU $USER && \
    chown -R $USER:$USER /app
```
Next we set a `USER` environment variable that we will use
throughout the Dockerfile. This next step is probably the one that is most often missed in
example Dockerfiles that you find on the interwebs. After specifying the `WORKDIR`, which will create the directory, we create a privileged user and group with the `useradd` command. Using the `-m` flag we are creating a home director for this user. This is important since many python packages may put data files in the user's home directory by default so we need it to exist. In this same step (because we want to keep the number of layers to a minimum) we also give this user ownership over the `WORKDIR`. This is important so that we don't end up with any runtime errors if our code writes and files to this directory. Note we are just creating the user and group here, we are not setting the user yet. Its generally good to install packages (especially system level packages) using the default root user and to set the privileged user at the end of the Dockerfile.

### Separating dependencies from source code
```dockerfile
COPY src/requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt && \
    rm -f requirements.txt
```
 In a lot of samples online you see some junk like `COPY . .`. Don't do this. First of all you will likely be copying a bunch of stuff you won't need, but even worse, you will be invalidating the cache any time you change any code. Since each layer is cached and since generally dependencies change way less often than the source code itself its best to separate dependencies from code as we do here. We just copy the `requirements.txt` file (remember those deterministic builds, use this requirements file as a real lock file using something like [pip-tools](https://github.com/jazzband/pip-tools), every dependency version should be fully specified). Keeping these layers separated like this will improve build times where you have a local cache and will also improve pull times on systems like Kubernetes.

In the next line we install our dependencies. Note that we are using the `--no-cache-dir` flag here, this will reduce the final docker image size since we are not creating a pip cache. We also clean up after ourselves by removing the requirements file since we don't need it to be in the final image.

### Move things that change often close to bottom of Dockerfile
```dockerfile
COPY --chown=$USER:$USER src/main.py /app

USER $USER
ENTRYPOINT ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8080"]
```
Ok, the dependencies are built, now we need to actually bring in the source code. Note that we use the `COPY --chown=user:group` syntax. This is because by default the copy command will copy the file with root ownership but we want our user to own the source code. 

Lastly, we set the user that (that only owns what is in this `WORKDIR`) and run our `ENTRYPOINT` script. There are some arguments as to whether you should use `ENTRYPOINT` or `CMD` for things like this but its a little harder to accidentally run the wrong thing with `ENTRYPOINT` than it is with `CMD` so I would recommend using `ENTRYPOINT`.

### Multi-stage builds for smaller images
In some cases, especially if your code requires compilers or other system level libraries it is best to use a [multi-stage build](https://docs.docker.com/build/building/multi-stage/)
where you install any system dependencies along with your python dependencies in the first stage and copy just the needed python environment into the second stage. Lets assume that
there a something in your code that requires a c-compiler when installing the python dependencies. Your Dockerfile may look something like
```dockerfile
FROM python:3.10.9-slim-bullseye as build

RUN apt-get update && apt-get install -y --no-install recommends \
    build-essential gcc 

WORKDIR /app
RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

COPY src/requirements.txt .
RUN pip install -r requirements.txt

FROM python:3.10.9-slim-bullseye as build

WORKDIR /app

RUN useradd -mrU $USER && \
    chown -R $USER:$USER /app

COPY --from=builder /opt/venv /opt/venv
COPY --chown=$USER:$USER src/main.py /app

USER $USER
ENV PATH="/opt/venv/bin:$PATH"
ENTRYPOINT ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8080"]
```
In the first stage we install our system level dependencies and our python dependencies. Note that we create a virtual environment here so we can copy over just our python dependencies into the second stage. The second stage creates our privileged user, copies over the virtual environment and sets the `PATH` variable to point there.  As a rule of thumb, any time you need extra "stuff" to setup your app that you won't need at runtime
you should use a two stage build and copy over the bare minimum of what will be needed.

# Conclusion
In this short post we have seen some best practices for building production level dockerfiles. There are several great resources out there that go into a lot more detail than this post but hopefully this will get you on your way to writing great Dockerfiles. Aforementioned great resources

* [Best practices from Snyk](https://snyk.io/blog/best-practices-containerizing-python-docker/)
* [Best practices from testdriven.io](https://testdriven.io/blog/docker-best-practices/)
* [Best practices from docker](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)


