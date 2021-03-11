# https://github.com/CircleCI-Public/circleci-dockerfiles/blob/master/android/images/api-29/Dockerfile

FROM openjdk:8-jdk-slim

LABEL maintainer="kan4649kan@gmail.com"

# make Apt non-interactive
RUN echo 'APT::Get::Assume-Yes "true";' > /etc/apt/apt.conf.d/90circleci \
  && echo 'DPkg::Options "--force-confnew";' >> /etc/apt/apt.conf.d/90circleci

ENV DEBIAN_FRONTEND=noninteractive

# Make sure PATH includes ~/.local/bin
# https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=839155
# This only works for root. The circleci user is done near the end of this Dockerfile
RUN echo 'PATH="$HOME/.local/bin:$PATH"' >> /etc/profile.d/user-local-path.sh

# man directory is missing in some base images
# https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=863199
RUN apt-get update \
  && mkdir -p /usr/share/man/man1 \
  && apt-get install -y \
    git xvfb apt \
    locales sudo openssh-client ca-certificates tar gzip parallel \
    net-tools netcat unzip zip bzip2 gnupg curl wget make

# Set timezone to UTC by default
RUN ln -sf /usr/share/zoneinfo/Etc/UTC /etc/localtime

# Use unicode
RUN locale-gen C.UTF-8 || true
ENV LANG=C.UTF-8

CMD ["/bin/sh"]

ARG sdk_version=sdk-tools-linux-4333796.zip
ARG android_home=/opt/android/sdk

# Download and install Android SDK
RUN mkdir -p ${android_home} && \
    curl --silent --show-error --location --fail --retry 3 --output /tmp/${sdk_version} https://dl.google.com/android/repository/${sdk_version} && \
    unzip -q /tmp/${sdk_version} -d ${android_home} && \
    rm /tmp/${sdk_version}

# Set environmental variables
ENV ANDROID_HOME ${android_home}
ENV ADB_INSTALL_TIMEOUT 120
ENV PATH=${ANDROID_HOME}/emulator:${ANDROID_HOME}/tools:${ANDROID_HOME}/tools/bin:${ANDROID_HOME}/platform-tools:${PATH}

RUN mkdir ~/.android && echo '### User Sources for Android SDK Manager' > ~/.android/repositories.cfg

RUN yes | sdkmanager --licenses && yes | sdkmanager --update

# Update SDK manager and install system image, platform and build tools
RUN sdkmanager \
  "tools" \
  "platform-tools" \
  "emulator"

RUN sdkmanager \
  "build-tools;29.0.3"

# API_LEVEL string gets replaced by m4
RUN sdkmanager "platforms;android-29"

RUN sdkmanager "cmake;3.6.4111459" "cmake;3.10.2.4988404" "ndk;17.2.4988734"

RUN apt-get update && apt-get install -y gradle && \
    cd /tmp && \
    gradle wrapper --gradle-version 4.10.1 --distribution-type all && \
    ./gradlew wrapper --gradle-version 4.10.1 --distribution-type all && \
    # rm -rf .gradle gradle  gradlew  gradlew.bat && \
    cd .. && \
    sudo rm -rf /var/lib/apt/lists/*
