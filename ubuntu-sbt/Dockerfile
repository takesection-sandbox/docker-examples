FROM ubuntu
MAINTAINER Pigumer Group Inc.
RUN apt-get update && \
    apt-get upgrade -y && \
    apt-get install zip openjdk-8-jre-headless curl -y && \
    echo 'curl -s "https://get.sdkman.io" | bash' >> /setup.sh && \
    echo 'source "/root/.sdkman/bin/sdkman-init.sh"' >> /setup.sh && \
    echo 'sdk install sbt' >> /setup.sh && \
    echo 'sdk install scala' >> /setup.sh && \
    chmod +x /setup.sh && \
    /bin/bash -c /setup.sh
