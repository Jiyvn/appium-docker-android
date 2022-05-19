FROM ubuntu:bionic-20220427

ENV DEBIAN_FRONTEND=noninteractive

ENV SDK_VERSION=commandlinetools-linux-8092744_latest \
    ANDROID_BUILD_TOOLS_VERSION=31.0.0 \
    APPIUM_VERSION=next \
    ATD_VERSION=1.2

#=============
# Set WORKDIR
#=============
WORKDIR /root

#==================
# General Packages
#------------------
# openjdk-8-jdk
#   Java
# ca-certificates
#   SSL client
# tzdata
#   Timezone
# zip
#   Make a zip file
# unzip
#   Unzip zip file
# curl
#   Transfer data from or to a server
# wget
#   Network downloader
# libqt5webkit5
#   Web content engine (Fix issue in Android)
# libgconf-2-4
#   Required package for chrome and chromedriver to run on Linux
# xvfb
#   X virtual framebuffer
# gnupg
#   Encryption software. It is needed for nodejs
# salt-minion
#   Infrastructure management (client-side)
#==================
RUN apt-get -qqy update && \
    apt-get -qqy --no-install-recommends install \
    openjdk-8-jdk \
    ca-certificates \
    tzdata \
    zip \
    unzip \
    curl \
    wget \
    libqt5webkit5 \
    libgconf-2-4 \
    xvfb \
    gnupg \
    salt-minion \
  && rm -rf /var/lib/apt/lists/*

#===============
# Set JAVA_HOME
#===============
#RUN echo '#!/bin/bash' >> /etc/profile.d/JAVA_HOME.sh && chmod +wx /etc/profile.d/JAVA_HOME.sh
#RUN export JAVA_HOME=$(echo /usr/lib/jvm/java-8-openjdk-*/jre) && \
#    echo 'export JAVA_HOME="'$JAVA_HOME'"' >> /etc/profile.d/JAVA_HOME.sh && \
#    echo 'export PATH='$PATH:$JAVA_HOME/bin >> /etc/profile.d/JAVA_HOME.sh && \
#    /bin/bash -c 'source /etc/profile'
#
#RUN echo ls -lrt $(which java) && sleep 10
#RUN echo "get java link:"
#RUN echo ls -lrt $(which java) | awk 'NR==1 {print $NF}' && sleep 10
#RUN echo "get java final link:"
#RUN echo $(ls -lrt $(ls -lrt $(which java) |awk 'NR==1 {print $NF}') |awk 'NR==1 {print $NF}') && sleep 10
#RUN /bin/bash -c 'source /etc/profile' && echo $JAVA_HOME && sleep 10 && cat /etc/profile.d/JAVA_HOME.sh && sleep 10

#RUN echo "update-alternatives --list java:"
#RUN update-alternatives --list java && sleep 10
#RUN echo "update-alternatives --display java:"
#RUN update-alternatives --display java && sleep 10
#RUN echo "update-alternatives --query java:"
#RUN update-alternatives --query java && sleep 10
#RUN echo "update-alternatives --config java:"
#RUN update-alternatives --config java && sleep 10

ENV JAVA_HOME="/usr/lib/jvm/openjdk8/jre"
ENV PATH=$PATH:$JAVA_HOME/bin
RUN echo "which java :"
RUN which java && sleep 10
RUN mv $(echo /usr/lib/jvm/java-8-openjdk-*/) /usr/lib/jvm/openjdk8 && sleep 10
#RUN echo "ls /usr/lib/jvm/openjdk8/jre :"
#RUN ls /usr/lib/jvm/openjdk8/jre && sleep 10
#RUN echo "update-alternatives --remove java:"
#RUN update-alternatives --remove-all java && sleep 10
#RUN echo "which java :"
#RUN which java && sleep 10
#RUN echo "update-alternatives --install java:"
RUN update-alternatives --install /usr/bin/java java $JAVA_HOME/bin/java 1081 \
    --slave /usr/share/man/man1/java.1.gz java.1.gz $JAVA_HOME/man/man1/java.1.gz \
     && sleep 10
#RUN echo "which java :"
#RUN which java && sleep 10
#RUN echo "update-alternatives --display java:"
#RUN update-alternatives --display java && sleep 10
#RUN echo "update-alternatives --query java:"
#RUN update-alternatives --query java && sleep 10

#=====================
# Install Android SDK
#=====================
ARG ANDROID_PLATFORM_VERSION="android-29"
ENV ANDROID_HOME=/root

RUN wget -O tools.zip https://dl.google.com/android/repository/${SDK_VERSION}.zip && \
    unzip tools.zip && rm tools.zip && \
    chmod a+x -R $ANDROID_HOME && \
    chown -R root:root $ANDROID_HOME

RUN mv cmdline-tools latest && mkdir cmdline-tools && mv latest cmdline-tools/latest

ENV PATH=$PATH:$ANDROID_HOME/cmdline-tools/latest:$ANDROID_HOME/cmdline-tools/latest/bin

# https://askubuntu.com/questions/885658/android-sdk-repositories-cfg-could-not-be-loaded
RUN mkdir -p ~/.android && \
    touch ~/.android/repositories.cfg && \
    echo y | sdkmanager "platform-tools" && \
    echo y | sdkmanager "build-tools;$ANDROID_BUILD_TOOLS_VERSION" && \
    echo y | sdkmanager "platforms;$ANDROID_PLATFORM_VERSION"

ENV PATH=$PATH:$ANDROID_HOME/platform-tools:$ANDROID_HOME/build-tools

#====================================
# Install latest nodejs, npm, appium
# Using this workaround to install Appium -> https://github.com/appium/appium/issues/10020 -> Please remove this workaround asap
#====================================
RUN curl -sL https://deb.nodesource.com/setup_17.x | bash && \
    apt-get -qqy install nodejs && \
    npm install -g appium@${APPIUM_VERSION} --unsafe-perm=true --allow-root && \
    exit 0 && \
    npm cache clean && \
    apt-get remove --purge -y npm && \
    apt-get autoremove --purge -y && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    apt-get clean

ARG DRIVER="uiautomator2,xcuitest"
RUN echo $DRIVER | xargs -d, -L 1 |xargs -L 1 appium driver install

#================================
# APPIUM Test Distribution (ATD)
#================================
RUN wget -nv -O RemoteAppiumManager.jar "https://github.com/AppiumTestDistribution/ATD-Remote/releases/download/${ATD_VERSION}/RemoteAppiumManager-${ATD_VERSION}.jar"

#==================================
# Fix Issue with timezone mismatch
#==================================
ENV TZ="US/Pacific"
RUN echo "${TZ}" > /etc/timezone

#===============
# Expose Ports
#---------------
# 4723
#   Appium port
# 4567
#   ATD port
#===============
EXPOSE 4723
EXPOSE 4567

#====================================================
# Scripts to run appium and connect to Selenium Grid
#====================================================
COPY entry_point.sh \
     generate_config.sh \
     wireless_connect.sh \
     wireless_autoconnect.sh \
     /root/

RUN chmod +x /root/entry_point.sh && \
    chmod +x /root/generate_config.sh && \
    chmod +x /root/wireless_connect.sh && \
    chmod +x /root/wireless_autoconnect.sh

#========================================
# Run xvfb and appium server
#========================================
CMD /root/wireless_autoconnect.sh && /root/entry_point.sh
