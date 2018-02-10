#docker build --build-arg node_version=6.11.5 --build-arg npm_version=3.10.10 --build-arg ionic_version=2.2.3 --build-arg cordova_version=6.5.0 --build-arg android_target_sdk=23 -t "ionic-android" . 
FROM     ubuntu:16.04
LABEL maintainer="bitard [dot] michael [at] gmail [dot] com"

ARG node_version=8.9.3
ARG npm_version=5.6.0
ARG ionic_version=3.19.0
ARG cordova_version=9.0.0
ARG android_target_sdk=26

ENV DEBIAN_FRONTEND=noninteractive \
    ANDROID_HOME=/opt/android-sdk-linux \
    NODE_VERSION=${node_version} \
    NPM_VERSION=${npm_version} \
    IONIC_VERSION=${ionic_version} \
    CORDOVA_VERSION=${cordova_version} \
    GRADLE_VERSION=4.3.1

# Install basics
RUN apt-get update &&  \
    apt-get install -y git wget curl unzip ruby ruby-dev gcc make && \
    curl --retry 3 -SLO "http://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.tar.gz" && \
    tar -xzf "node-v$NODE_VERSION-linux-x64.tar.gz" -C /usr/local --strip-components=1 && \
    rm "node-v$NODE_VERSION-linux-x64.tar.gz" && \
    npm install -g npm@"$NPM_VERSION" && \
    npm install -g cordova@"$CORDOVA_VERSION" ionic@"$IONIC_VERSION" && \
    npm cache clear --force && \
#    gem install sass && \
    git config --global user.email "you@example.com" && \
    git config --global user.name "Your Name" && \
    ionic start myApp sidemenu --no-interactive
#ANDROID
#JAVA

# install python-software-properties (so you can do add-apt-repository)
RUN apt-get update && apt-get install -y -q python-software-properties software-properties-common  && \
    add-apt-repository ppa:webupd8team/java -y && \
    echo oracle-java8-installer shared/accepted-oracle-license-v1-1 select true | /usr/bin/debconf-set-selections && \
    apt-get update && apt-get -y install oracle-java8-installer

#ANDROID STUFF
RUN echo ANDROID_HOME="${ANDROID_HOME}" >> /etc/environment && \
    dpkg --add-architecture i386 && \
    apt-get update && \
    apt-get install -y --force-yes expect ant wget zipalign libc6-i386 lib32stdc++6 lib32gcc1 lib32ncurses5 lib32z1 qemu-kvm kmod && \
    apt-get clean && \
    apt-get autoclean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Install Android SDK
RUN cd /opt && \
    mkdir ${ANDROID_HOME} && \
    wget --output-document=android-sdk-linux.zip --quiet https://dl.google.com/android/repository/sdk-tools-linux-3859397.zip && \
    unzip -d ${ANDROID_HOME} android-sdk-linux.zip && \
    rm -f android-sdk-linux.zip && \
    chown -R root. /opt

# Install Gradle
RUN wget https://services.gradle.org/distributions/gradle-"$GRADLE_VERSION"-bin.zip && \
    mkdir /opt/gradle && \
    unzip -d /opt/gradle gradle-"$GRADLE_VERSION"-bin.zip && \
    rm -rf gradle-"$GRADLE_VERSION"-bin.zip

# Setup environment

ENV PATH ${PATH}:${ANDROID_HOME}/tools/bin:${ANDROID_HOME}/platform-tools:/opt/tools:/opt/gradle/gradle-"$GRADLE_VERSION"/bin

# Install sdk elements
COPY tools /opt/tools

RUN echo y | sdkmanager "platform-tools" "tools" "build-tools;${android_target_sdk}.0.2" "platforms;android-${android_target_sdk}"  "extras;android;m2repository" "extras;google;m2repository"

#RUN ["/opt/tools/android-accept-licenses.sh", "android update sdk --all --no-ui --filter platform-tools,tools,build-tools-23.0.2,android-23,extra-android-support,extra-android-m2repository,extra-google-m2repository"]
#RUN unzip ${ANDROID_HOME}/temp/*.zip -d ${ANDROID_HOME}

# Test First Build so that it will be faster later
RUN cd myApp && \
    ionic cordova build android --prod --no-interactive --release && \
    ionic cordova plugin add cordova-plugin-crosswalk-webview && \
    ionic cordova build android --prod --no-interactive --release

WORKDIR myApp
EXPOSE 8100 35729
CMD ["ionic", "serve"]
