# data-engineer-docker-workshop
Workshop codespace for docker
## What is Docker
Docker is a containerization software that allows us to isolate software in a similar way to virtual machines but in a much leaner way.

A Docker image is a snapshot of a container that we can define to run our software, or in this case our data pipelines. By exporting our Docker images to Cloud providers such as AWS or Google Cloud Platform we can run our containers there.

Docker allows you to completely run an application and its service dependencies isolated or in isolation from the host machine. Whatever is changed in the dockerized app or service has no affect on the host machine. A Docker image is stateless


## Why Docker?
Docker provides the following advantages:

 - Reproducibility: Same environment everywhere
 - Isolation: Applications run independently
 - Portability: Run anywhere Docker is installed

They are used in many situations:
 - Integration tests: CI/CD pipelines
 - Running pipelines on the cloud: AWS Batch, Kubernetes jobs
 - Spark: Analytics engine for large-scale data processing
 - Serverless: AWS Lambda, Google Functions

## Basic Docker Commands
Check Docker version
```
docker --version
```
Run a simple container
```
docker run hello-world
```
Run something more complex:
```
docker run ubuntu
```
Nothing happens. Need to run it in  ``` -it ```  or interactive terminal mode:
```
docker run -it ubuntu
```
I dont have ``` python ``` there so lets install it:
```
apt update && apt install python3
python3 -V
```

## Stateless Containers
Important: Docker containers are stateless - any changes done inside a container will NOT be saved when the container is killed and started again. It does NOT maintain state. 

When you exit the container and use it again, the changes are gone:

```
docker run -it ubuntu
python3 -V
```
This is good, because it doesn't affect your host system. Let's say you do something crazy like this:
```
docker run -it ubuntu
rm -rf / # don't run it on your computer!
```
Next time we run it, all the files are back. 
Now let's run another image; an image containing Debian with python version 3.13.11
```
docker run -it python:3.13.11
```
Lets exit out of it
```
exit()
```
Now let's run a smaller image; an image containing Debian with python version 3.13.11
```
docker run -it python:3.13.11-slim
exit()
```

Now let's run a smaller image; an image containing Debian with python version 3.13.11 but using bash as the entry point for command line
```
docker run -it --entrypoint=bash python:3.13.11-slim
```

## Other things
You can choose to create files in the image, but just know they will not be saved when you exit the image. However, you should NEVER create files directly in the containerized image. It is not best practice.
### Writing to a file
```
echo 123 > file
```
This writes the text 123 into a file named file, creating it if it doesn’t already exist. If file already exists, it gets overwritten.
### Checking out or viewing the file
```
cat file 
```
This opens the file


## Managing Containers
But, this is not completely correct. The state of the version that was used is saved somewhere. Lets see all the running containers we have executed or used in the past. We can see stopped containers:

```
docker ps -a
```
We can restart one of them, but we won't do it, because it's not a good practice. They take space, so let's delete them:

```
# Lets get the Ids for all the containers
docker ps -aq
## You could delete container individually
docker rm [container id]
## Alternatively, delete all containers with their ids
docker rm `docker ps -aq`
```
Next time we run something, we add --rm:
```
docker run -it --rm ubuntu
```

## Different Base Images
There are other base images besides hello-world and ubuntu. For example, Python:

```
docker run -it --rm python:3.9.16
# add -slim to get a smaller version
```
This one starts python. If we want bash, we need to overwrite entrypoint:
```
docker run -it \
    --rm \
    --entrypoint=bash \
    python:3.9.16-slim
```

## Volumes
So, we know that with docker we can restore any container to its initial state in a reproducible manner. But what about data? A common way to do so is with volumes.

Let's create some data in test:
```
mkdir test
cd test
touch file1.txt file2.txt file3.txt
echo "Hello from host" > file1.txt
cd ..
```
Now let's create a simple script test/list_files.py that shows the files in the folder:
```
from pathlib import Path

current_dir = Path.cwd()
current_file = Path(__file__).name

print(f"Files in {current_dir}:")

for filepath in current_dir.iterdir():
    if filepath.name == current_file:
        continue

    print(f"  - {filepath.name}")

    if filepath.is_file():
        content = filepath.read_text(encoding='utf-8')
        print(f"    Content: {content}")
```
Now let's map this to a Python container:
```
# Mapping the test directory to app/test directory in the image `python:3.9.16-slim`
docker run -it \
    --rm \
    -v $(pwd)/test:/app/test \
    --entrypoint=bash \
    python:3.9.16-slim

```
Inside the container, run:
```
cd /app/test
ls -la
cat file1.txt
python list_files.py
```
You'll see the files from your host machine are accessible in the container!