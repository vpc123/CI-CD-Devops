FROM registry.cn-hangzhou.aliyuncs.com/ubuntu/ubuntu:hdpOK

LABEL maintainer="Bo-Yi Wu <appleboy.tw@gmail.com>"

ENV FLUTTER_HOME ${HOME}/flutter
ENV FLUTTER_VERSION 1.0.0-stable

RUN apt-get update \
  && apt-get install -y libglu1-mesa git curl unzip wget xz-utils lib32stdc++6 \
  && apt-get clean

WORKDIR /