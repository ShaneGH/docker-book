# Docker stuff

* Docker is images + containers
    * An image is a piece of a layered file system
    * A container is created fom an image and can be run
* Docker is a client server app. The command line client sends commands to the service which builds images

# `docker info`

Get info on the running docker service

# `docker build`

Build a `Dockerfile` line by line. If one line fails, the previous one will have been built, so you can still start the previous line as a container and debug it. Uses `.dockerignore` like a gitignore when using `ADD` and `COPY` instructions.

* Last arg, context. The context is a directory which `docker build` will run from, and cannot access files any higher.
* `-t` tag image (`-t imagename:tagname`)
* `--no-cache` Rebuild from scratch, even if previous iterations were cached
    * `COPY` and `ADD` instructions are never cached
    * Trick to control cache: `ENV REFRESHED_AT 2020-01-01`
* `--build-arg`, specify args required from `ARG` `Dockerfile` instruction

# Dockerfile

Each line in a docker file is a new instruction to help build an image. After the command is run, `docker build` will commit the image. So, each line is like a separate commit

* `FROM ubuntu:16.04`, base image => name:tag
* `MAINTAINER Shane Connon "shane.connon@email.com"`
* `RUN apt-get update; apt-get install -y nginx`, run commands as part of image build
* `CMD`, run a command when the container starts
    * only one per dockerfile
    * will be overridden by command specified in `docker run`
    * if used with `ENTRYPOINT`, will be passed as args to `ENTRYPOINT`
* `ENTRYPOINT`, like `CMD` but stronger
    * only one per dockerfile
    * if a command is specified in `docker run`, it will be passed as args to `ENTRYPOINT`
* `EXPOSE 80`, expose port 80 from within the container
* `ENV hello you`, creates the environment variable `hello` with value `you`
    * You can use env variables in dockerfile with `$hello`
* `ONBUILD`, trigger in parent image to say that the child image was built
    * e.g. waiting on child container to populate tests: `ONBUILD RUN /my_code/tests.cmd`
* `WORKDIR` change working dir in image (like `cd`)
* `USER` change user in image
* `VOLUME`, create a volume
    * volumes are mounted drives. They can come from the host machine and can be shared by multiple containers
    * are independant of container lifecycle
* `ADD` add a file from host machine to image (syntax can be tricky on windows)
    * Ignores files in `.dockerignore`
    * will invalidate the image cache (see `docker build --no-cache`)
    * e.g. `ADD /file.txt /file.txt`
    * e.g. `ADD /files/ /files/`
    * e.g. `ADD files.tar.gz /files/`, will unzip archive
* `COPY` same as `ADD` but without compression support
* `LABEL` add metadata to an image which will show in `docker inspect`
    * `LABEL "key"="value"`
* `ARG` specify arg which can be passed in during `docker build --build-arg`.
    * `ARG arg1`, specify mandatory arg
    * `ARG arg1=hello`, specify optional arg with default value
    * There are some default args on all images (e.g. `HTTP_PROXY`)
* `SHELL` specify the shell. Defaults provided for all envs
* `HEALTHCHECK`, like heartbeat. Inspect health using `docker inspect`
    * `--interval=10s` healthcheck interval
    * `--timeout=1m`
    * `--retries=5`
    * `CMD /dosomething`, should exit with a 1 or a 0
    * `NONE`, disable healthcheck from parent image

# .dockerignore

Ignore files in `ADD` and `COPY` instructions. Uses go's filepath for globbing

# `docker history`

View the layers of an image

# `docker run`

Create a container from an image and run it. Multiple containers can be created from the same image

* Arg 0, image name to run
* Arg 1, command to send to container (see `Dockerfile` section)
* `-i` keep stdin connected to your command line
* `-t` open a pseudo-tty shell. Use with `-i`
* `-d` detach and run in the background
* `--name` name the container
* `--log-driver` send logs somewhere else
* `--restart=always` restart the container if it crashes
* `-p` assign a specific port from the host machine to the container (-p hostPort:containerPort)
* `-P` assign random ports from the host machine to the container
* `-w` override `WORKDIR` from `Dockerfile`
* `-u` the user to run as in the container (default root)

# `docker ps`

Show running containers + details

* `-a` show all, not just running
* `-l` last container only

# `doker rmi`

Remove an image

# `doker rm`

Remove a container

# `docker stop`

Stop a container. `docker kill` is a more aggressive version. (sends a different stop signal)

# `docker start`

Start a container Container must have been created using `docker run`. Also `docker restart`

# `docker attach`

Attach to a container running in the background

# `docker logs`

Get logs from a container. Default last few logs.

* `-f` all logs
* `-t` include timestamp

# `docker top`

View processes running inside a container

# `docker stats`

View stats, such as CPU, memory, etc...

* Each arg to this command is a container name to view

# `docker exec`

Run a process inside a running container

* e.g. `docker exec -d my_container touch /etc/new_file`
    * `-d` is in the background
    * `touch /etc/new_file` creates a new file
* e.g. Shell into running container: `docker exec -t -i my_container /bin/bash`
    * see `docker run` for `-t -i`
    * `bin/bash` run a shell

# `docker inspect`

Inspect the status and metadata of a container. 
* Add metada with `LABEL` `Dockerfile` instruction
* Add status with `HEALTHCHECK` `Dockerfile` instruction

# `docker images`

Show images on local machine

* `--format`, output format

# `docker search`

Seach dockerhub

# `docker port`

Display ports used by docker
