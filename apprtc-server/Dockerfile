FROM --platform=linux/amd64 ubuntu:bionic
LABEL maintainer="Hyunjoon Ko (linus@vlending.co.kr)"

EXPOSE 8080 8089 3478 3033 49160-49200

WORKDIR /

ENV GAE_VER 1.9.74
ENV GOLANG_VER 1.19
ENV LIBEVENT_VER 2.1.8
ENV COTURN_VER 4.5.0.7

ENV PUBLIC_IP 127.0.0.1

# install python 3
RUN apt-get update -y
RUN apt-get install -y software-properties-common
RUN apt-add-repository ppa:deadsnakes/ppa
RUN apt-get install -y python3.7
RUN apt-get install -y python3-pip

# Deps
RUN apt-get install -y build-essential vim git curl wget unzip python-pil python-webtest libssl-dev openjdk-8-jdk && \
    rm -rf /usr/lib/python3.7/dist-packages/supervisor* && \
    pip3 install supervisor requests && \
    pip3 install git+https://github.com/bendikro/supervisord-dependent-startup

# NodeJS
RUN curl -sL https://deb.nodesource.com/setup_14.x | bash -
RUN apt-get install -y nodejs

# Golang https://go.dev/dl/go1.8.3.linux-amd64.tar.gz
ENV GOLANG_TAR go$GOLANG_VER.linux-amd64.tar.gz
RUN wget https://go.dev/dl/$GOLANG_TAR
RUN tar -C /usr/local -xzf $GOLANG_TAR
ENV PATH $PATH:/usr/local/go/bin

# Google App Engine
ENV GAE_ZIP google_appengine_$GAE_VER.zip
RUN wget https://storage.googleapis.com/appengine-sdks/featured/$GAE_ZIP
RUN unzip $GAE_ZIP -d /usr/local
ENV PATH $PATH:/usr/local/google_appengine

# Coturn server
RUN wget https://github.com/libevent/libevent/releases/download/release-$LIBEVENT_VER-stable/libevent-$LIBEVENT_VER-stable.tar.gz
RUN tar xvfz libevent-$LIBEVENT_VER-stable.tar.gz
WORKDIR libevent-$LIBEVENT_VER-stable
RUN ./configure && make && make install
WORKDIR /
RUN wget http://turnserver.open-sys.org/downloads/v$COTURN_VER/turnserver-$COTURN_VER.tar.gz
RUN tar xvfz turnserver-$COTURN_VER.tar.gz
WORKDIR turnserver-$COTURN_VER
RUN ./configure && make && make install
RUN turnadmin -a -u ninefingers -r apprtc -p youhavetoberealistic
COPY turnserver.conf /etc/turnserver.conf

# AppRTC
WORKDIR /
RUN git clone https://github.com/webrtc/apprtc.git

WORKDIR apprtc

RUN export GOPATH=$HOME/goWorkspace
RUN mkdir -p $GOPATH/src

RUN ln -s /apprtc/src/collider/collider $GOPATH/src
RUN ln -s /apprtc/src/collider/collidermain $GOPATH/src
RUN ln -s /apprtc/src/collider/collidertest $GOPATH/src
RUN go install github.com/tkausch/collidermain@latest

RUN npm install -g npm
RUN npm install -g grunt-cli

RUN npm install
RUN grunt build

WORKDIR /usr/app

RUN npm install express
COPY ice.js /
COPY constants.py /apprtc/out/app_engine/constants.py

# Clean up APT when done.
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

COPY apprtc_supervisord.conf /

COPY run.sh /
RUN chmod +x /run.sh
CMD /run.sh
