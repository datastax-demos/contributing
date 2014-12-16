FROM       datastaxdemos/datastax-enterprise:stable
MAINTAINER Joaquin Casares <joaquin@datastax.com>

# install git to start the service
RUN apt-get update && \
    apt-get install -y \
        git-core \
        python-dev \
        python-pip && \
    rm -rf /var/lib/apt/lists/*

# install node components
RUN apt-get update && \
    apt-get install -y \
        curl && \
    curl -sL https://deb.nodesource.com/setup | sudo bash - && \
    apt-get install -y \
        nodejs && \
    npm install pm2 -g && \
    rm -rf /var/lib/apt/lists/*

# setup github ssh keys
RUN mkdir -p /root/.ssh
COPY src/coffice.key /root/.ssh/
COPY src/coffice.key.pub /root/.ssh/
COPY src/config /root/.ssh/
RUN ssh-keyscan github.com >> /root/.ssh/known_hosts
RUN chmod -R 600 /root/.ssh

# clone demo source
RUN git clone git@coffice:riptano/coffice.git /root/coffice

# copy start scripts and set start CMD
COPY bin/start-coffice /usr/local/bin/start-coffice
ENTRYPOINT ["start-coffice"]

EXPOSE 3000
