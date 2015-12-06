# Kafka Development Environment

This repository gives you the ability to create a local [Kafka][kafka] cluster
for local development and experimentation.

##  Setup

### Linux

0. Install `docker` via your distribution's package manager.
0. Download and install [Docker Compose][docker-compose].
0. Download and install the [Confluent Platform][confluent-platform].

### OS X

NB: You will need to have [Homebrew][homebrew] installed.

0. Install [Docker Machine][docker-machine]

    brew install docker-machine

0. Install [Kafka][kafka]

    brew install kafka

0. Install [Zookeeper][zookeeper]

    brew install kafka

0. You will need to carry out this [routing stepup step](./OSX-Routing.md) now
   and every time you reload your routing table or restart the OS.

## Running

### Linux

0. You'll need to create local directories (under `/tmp/docker`) that are linked
   to directories internally used by the containers. Set up all associated local
   volumes/directories by running this command:

    mkdir -p `grep /tmp/docker docker-compose.yml | cut -d' ' -f6 | cut -d':' -f1 | sort`

### General

0. Start all the needed containers by running:

    docker-compose up -d

0. Running `docker-compose ps` or `docker ps` should show at least 1 Zookeeper
   container and 1 or 3 Kafka containers.

##  Walkthroughs

These walkthroughs are aimed at getting you familiar with Kafka. They are
written with the aim of getting different audiences (developers, system
administrators) comfortable using Kafka.

0. [**Basic Walkthrough**](./walkthroughs/basic_walkthrough/README.md) - Aimed at everyone and *everyone* should go through this.
0. **Ruby Walkthrough** - TODO
0. **Clojure Walkthrough** - TODO

## Container controls

You can control the running "composed" containers as follows:

* Stop all the containers:

    docker-compose stop

* Remove all the containers:

    docker-compose rm`

##  Notes

0. Currently the Zookeeper and Kafka containers use volumes that are on the
   local filesystem for Linux, but for OS X are inside the `boot2docker` vm
   (type `boot2docker ssh` and explore `/tmp/docker`). These volumes will
   survive container restarts and should be deleted (and recreated for Linux
   users) if you want your containers to start with clean volumes.
0. If you need to repoint the `docker-compose.yml` soft link, please shutdown
   and remove the current configuration first by running:

    docker-compose stop && docker-compose rm

##  TODO

0. Add a monitoring docker (Already have the perfect candidate)
0. Expose/link the configuration directories of Zookeeper and Kafka containers
   the same way the logs and data directories are exposed. This will allow
   people to further experiment with various configurations.
0. Explore using a data/volume container instead of volumes linked to local disk.

[confluent-platform]: http://www.confluent.io/developer#download
[docker-compose]: http://docs.docker.com/compose/install/
[docker-toolbox]: https://www.docker.com/docker-toolbox
[homebrew]: http://brew.sh
[kafka]: http://kafka.apache.org
[zookeeper]: http://zookeeper.apache.org/
