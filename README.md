## Guide for Contributing to the DataStax Demo Portal

Included in this repository is the shell of an active DataStax Demo Portal demo.

The only missing file is `connected-office/src/coffice.key`, which is a two-way
Github repository "deploy key".

This guide is intended for users who have no familiarity with Docker.

## Setup

You'll need [Docker](http://docker.com).

### Ubuntu

```bash
curl -sSL https://get.docker.com/ubuntu/ | sudo sh
```
    
### OS X

You can install Docker using [Homebrew](http://brew.sh/)
    
```bash
brew update; brew install docker boot2docker docker-completion
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
* Ensure you also set the MAINTAINER variable in case there are any issues
on our end with your script.

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
* Always `rm -rf /var/lib/apt/lists/*` if you've done an `apt-get update` to
save on layer size.

```ruby
RUN mkdir -p /root/.ssh
```

* All docker commands will happen as the root user, if this matters for your
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
    
* Use this loop, or similar loop to wait for the native transport to start.

```bash
cqlsh -f schema.cql ${IP}
```

* Use a similar command to load your schema from a file.

```bash
nohup python coffice/scripts/sensor_data/load_snacks.py > /root/load_snacks.out 2>&1 &
```

* Use `nohup $(command) &` to start a process that will run in the
background.
* Redirect your stdout and stderr to a file using `> output.log 2>&1` for easier
debugging.

```bash
# keep this script running for docker
echo "Stay alive..."
while :; do sleep 1; done
```

* An infinite loop is required to keep the Docker container active. If not, the
Docker container will stop as soon as the script returns.

## Running a Docker Container

```bash
docker pull ubuntu:trusty
docker build -t="ouruser/demo:v1" .
docker run -d --net host ouruser/demo:v1 ${IP_LIST}
```

* `ubuntu:trusty` can be substituted for the
[image](https://registry.hub.docker.com/) used by the Dockerfile.

* `ouruser/demo:v1` will be the `<organization>/<name>:<version>` of the build.
* The trailing `.` is used to signify the current path to the Dockerfile.

* `-d` will enable daemon mode and run the container in the background.
* `--net hose` will use the host network stack inside the container.
* `${IP_LIST}` will be the comma-delimited list of DataStax Enterprise nodes
that will be passed to your `ENTRYPOINT`.

## Additional Docker Commands

```bash
docker ps
```

* Will report all running containers and container ids.

```bash
docker rm -f <container_id>
```

* Will kill an active container and remove it from your system.
