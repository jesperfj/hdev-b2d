# Auto-mount host folder in boot2docker

This project allows you to build a custom boot2docker.iso that will auto-mount a virtual box shared folder called `dev-home` on `/dev-home` when boot2docker starts up (so you don't have to run mount manually).

I take zero credit for this work. Read [Matthias Kadenbach's excellent post](https://medium.com/boot2docker-lightweight-linux-for-docker/boot2docker-together-with-virtualbox-guest-additions-da1e3ab2465c) to understand why this is painful and how this solution came about.

I customized Matthias' image because I did not want to mount all of `/Users` and because I wanted to experiment with running as non-root in the docker container. But that's literally all the diff.

## Building the image

Clone this repo and cd into the directory. Make sure boot2docker is running. Then:

    $ docker build -t mattes/boot2docker-vbga .

This will take a bit. Rebuilds will be faster because of docker's caching of intermediary containers / FS layers. The command will build an iso and store it inside the container file system. To get it out, I use the same trick as Matthias. In one terminal, I run:

    $ docker run -t -i mattes/boot2docker-vbga bash

then, in another terminal, I pick up the container id:

```
$ docker ps
CONTAINER ID        IMAGE                            COMMAND                CREATED             STATUS              PORTS               NAMES
d182bce465ec        mattes/boot2docker-vbga:latest   bash                   8 seconds ago       Up 4 seconds                            mad_meitner
```

(you'll see a different ID). Then I use docker's `cp` command to copy out the image:

    $ docker cp d182bce465ec:/boot2docker.iso /tmp/boot2docker.iso

Now stop boot2docker on your host system, make a backup of your current iso and copy the new one in place:

```
$ boot2docker stop
$ cp ~/.boot2docker/boot2docker.iso ~/.boot2docker/boot2docker.works
$ mv /tmp/boot2docker.iso ~/.boot2docker/boot2docker.iso
```

Before you start boot2docker again, set up a virtual box shared folder named "dev-home":

```
$ VBoxManage sharedfolder add boot2docker-vm --name dev-home --hostpath /path-of-your-choice
```

A dev-home is a top-level home folder for all your development projects where each project will be the home folder of the docker user. This allows you to hop into a docker container and have the home folder of the user live on your host system (and therefore be persistent across docker runs).

See [the hdev project](https://github.com/jesperfj/hdev) for an example Docker container that uses this configuration.
