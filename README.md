## Guide for Contributing to the DataStax Demo Portal

By contributing demos to the DataStax Demo Portal, you are allowing multiple
teams within DataStax (pre-sales, post-sales, training, marketing, community)
and external to DataStax (partners, vendors, customers) to be
able to quickly and easily deploy DataStax Enterprise clusters that are being
utilized by your application. These applications will in turn be showcased in
front of potential clients, current customers, students, and conferences.

For internal teams, we will ideally be showcasing the overall power
of DataStax Enterprise and Apache Cassandra while highlighting individual
features or market verticals.

For partner teams, we will ideally be showcasing how your application
integrates with DataStax Enterprise in distinct scenarios.

Included in this repository is the shell of an active DataStax Demo Portal
Docker container for tutorial purposes. The container is, however, missing a
single file that blocks the container from
running locally. `connected-office/src/coffice.key` was removed for security
reasons.

This guide is intended for users who have no familiarity with Docker.

## Setup

[Docker](http://docker.com) containers are required for the DataStax Demo Portal
to integrate additional demos. Containers provide "an operating system–level
virtualization method for running multiple isolated Linux systems (containers)
on a single control host. The Linux kernel comprises cgroups for resource
isolation (CPU, memory, block I/O, network, etc.) that does not require
starting any virtual machines." [Wikipedia](http://en.wikipedia.org/wiki/LXC)
Find out more [here](https://www.docker.com/whatisdocker/).

Install Docker to get started.

### Ubuntu

```bash
curl -sSL https://get.docker.com/ubuntu/ | sudo sh
```
    
### OS X

Docker can be installed using [Homebrew](http://brew.sh/)
    
```bash
brew update
brew tap homebrew/completions
brew install docker boot2docker docker-completion
```
    
or use the [official documentation](https://docs.docker.com/installation/mac/).

### Other OS

All other OS guides can be found [here](http://docs.docker.com/installation/).


## Key Files

### Dockerfile

```ruby
FROM       datastaxdemos/datastax-enterprise:stable
MAINTAINER Joaquin Casares <joaquin@datastax.com>
```

* The FROM variable in this case shows this images is a child of the
`datastaxdemos/datastax-enterprise:stable` private image. A good alternative
would be `ubuntu:trusty`. Other images can be found
[here](https://registry.hub.docker.com/).
* Ensure the MAINTAINER variable is set in case there are any issues with the
script.

```ruby
RUN apt-get update && \
    apt-get install -y \
        git-core \
        python-dev \
        python-pip && \
    rm -rf /var/lib/apt/lists/*
```
    
* Group all similar commands into as few distinct blocks as possible to save on
layer sizes. (Each layer will have to be downloaded on each demo launch thus
taking more time.)
* Always `rm -rf /var/lib/apt/lists/*` if `apt-get update` has been run to
save on layer size.

```ruby
RUN mkdir -p /root/.ssh
```

* All docker commands will happen as the root user, if this matters for the demo
environment.

```ruby
COPY src/coffice.key /root/.ssh/
```

* Keep all src material neatly organized into a `src/` directory.

```ruby
COPY bin/start-coffice /usr/local/bin/start-coffice
```

* Keep all binaries neatly organized into a `bin/` directory.
* Binaries can be any sort of executable script. Bash preferred for simple
scripts. Python preferred for complex scripts. Ultimately up to the maintainer.

```ruby
ENTRYPOINT ["start-coffice"]
```

* The command that the Docker container will run on start.

```ruby
EXPOSE 3000
```

* A list of all ports in a single line that will require public access.
* Because we use `--net host` this is a moot command, but good to know for our
backend infrastructure.

### bin/start-coffice

```bash
# grab local IP address and grab seedlist, if applicable
HOST=$(hostname -i)
ARG2="$1"
IP_LIST=${ARG2:-${HOST}}
IP=`echo $IP_LIST | cut -d',' -f1`
```

* The `ENTRYPOINT` will be sent a single argument with a comma-delimited list of
DataStax Enterprise IP addresses.
* If no argument is provided, please assume that DataStax Enterprise is running
on the same machine.

```bash
# wait for Cassandra's native transport to start
while :
do
    echo "SELECT bootstrapped FROM system.local;" | cqlsh ${IP} && break
    sleep 1
done
```
    
* Use this loop, or similar loop, to wait for the native transport to start.

```bash
cqlsh -f schema.cql ${IP}
```

* Use a similar command to load the schema from a file.

```bash
python coffice/scripts/sensor_data/load_snacks.py > /root/load_snacks.out 2>&1 &
```

* Use `$(command) &` to start a process that will run in the
background.
* Redirect the stdout and stderr to a file using `> application.out 2>&1` for
easier debugging.

```bash
# keep this script running for docker
echo "Stay alive..."
while :; do sleep 1; done
```

* An infinite loop is required to keep the Docker container active. If not, the
Docker container will stop as soon as the script returns.

### SCRIPT.*

* A [Github Markup](https://github.com/github/markup)
document should be included with each Docker container.
* Feel free to use any of the
[supported markups](https://github.com/github/markup#markups)
with matching file extensions.
* An overview, intended audience, intended market verticals, and script should
be provided.

### PARTNER-SCRIPT.*

* Required for partner demos, not DataStax demos.
* A [Github Markup](https://github.com/github/markup)
document should be included with each Docker container.
* Feel free to use any of the
[supported markups](https://github.com/github/markup#markups)
with matching file extensions.
* An overview, reason for DataStax Enterprise, intended audience,
intended market verticals, and script should be provided.
* This document should ideally mention why DataStax Enterprise is the ideal
choice for this demo, but feel free to focus primarily on selling the partner
application.

## Running a Docker Container

```bash
docker pull ubuntu:trusty
```

* `ubuntu:trusty` can be substituted for the
[image](https://registry.hub.docker.com/) used by the Dockerfile.

```bash
docker build --tag "ouruser/demo:v1" .
```

* `ouruser/demo:v1` will be the `<organization>/<name>:<version>` of the build.
* The trailing `.` is used to signify the current path to the Dockerfile.

```bash
docker run --detach --net host ouruser/demo:v1 ${IP_LIST}
```

* `--detach` will enable daemon mode and run the container in the background.
* `--net host` will use the host network stack inside the container.
* `${IP_LIST}` will be the comma-delimited list of DataStax Enterprise nodes
that will be passed to the Dockerfile's `ENTRYPOINT`.

## Additional Docker Commands

```bash
docker ps
```

* Will report all running containers and container ids.


```bash
ID=$(docker ps | awk '{if (NR==2) print $1}')
docker exec -it ${ID} bash
```

* Allows access into a running Docker container for manual probing and
interaction.
* The first line grabs the id from the command-line, assuming a single container
is running. If not, assign this variable manually.
* The second line uses `-it`, shorthand for `--interactive --tty`, and starts
the `bash` command inside of the container referenced by `${ID}`.

```bash
docker rm -f <container_id>
```

* Will kill an active container and remove it from the host system.

## Submitting New Demos

* Send an email to `demos@datastax.com` with either of the following options
(in order of preference):
    * a link to a public [Github](http://github.com) repository
    * a link to a (private or public) [gist](https://gist.github.com/)
    * a link to a private [Github](http://github.com) repository
        * add permissions for `joaquincasares` and `tupshin` to access the
        repository
    * a link to a [hub.docker.com](http://hub.docker.com) repository
        * add permissions for `joaquincasares` and `datastaxdemosuser` to access
        the repository
    * a tarball
* Code repositories preferred due to the added benefits of:
    * streamlined upgrades
    * simplified code management
    * point-in-time recovery
* Feel free to send us a [hub.docker.com](http://hub.docker.com) repository,
but do be aware any changes made to this container will be immediately 
available on our live system.
