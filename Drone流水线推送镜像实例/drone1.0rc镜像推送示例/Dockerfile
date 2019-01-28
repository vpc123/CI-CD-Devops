FROM registry.cn-hangzhou.aliyuncs.com/ubuntu/ubuntu:hdpOK

ENV FLUTTER_HOME ${HOME}/flutter
ENV FLUTTER_VERSION 1.0.0-stable

RUN apt-get update \
  && apt-get install -y libglu1-mesa git curl  \
  && apt-get clean

WORKDIR /