FROM swift:5.0.2 as builder
LABEL maintainer="IBM Swift Engineering at IBM Cloud"
LABEL Description="Build stage of Template Dockerfile that produces a Swift binary based on the swift:slim image."


# Default user if not provided
ARG bx_dev_user=root
ARG bx_dev_userid=1000

# Create user if not root
RUN if [ "$bx_dev_user" != root ]; then useradd -ms /bin/bash -u $bx_dev_userid $bx_dev_user; fi

# Install system level packages
RUN apt-get update && apt-get dist-upgrade -y && apt-get install -y sudo libssl-dev libcurl4-openssl-dev

# We can replace this port with what the user wants
EXPOSE 8080 1024 1025

# Add utils files
ADD https://raw.githubusercontent.com/IBM-Swift/swift-ubuntu-docker/master/utils/tools-utils.sh /swift-utils/tools-utils.sh
ADD https://raw.githubusercontent.com/IBM-Swift/swift-ubuntu-docker/master/utils/common-utils.sh /swift-utils/common-utils.sh
RUN chmod -R 555 /swift-utils

# Make password not required for sudo.
# This is necessary to run 'tools-utils.sh debug' script when executed from an interactive shell.
# This will not affect the deploy container.
RUN echo "$bx_dev_user ALL=NOPASSWD: ALL" > /etc/sudoers.d/user && \
    chmod 0440 /etc/sudoers.d/user

# Bundle application source & binaries
COPY . /swift-project

# Build application
WORKDIR /swift-project
RUN /swift-utils/tools-utils.sh build release

FROM swift:5.0.2-slim
LABEL maintainer="IBM Swift Engineering at IBM Cloud"
LABEL Description="Run stage of Template Dockerfile that extends the swift:slim image and contains a pre-built Swift application."

# Default user if not provided
ARG bx_dev_user=root
ARG bx_dev_userid=1000

# Create user if not root
RUN if [ $bx_dev_user != "root" ]; then useradd -ms /bin/bash -u $bx_dev_userid $bx_dev_user; fi

# Install system level packages
# RUN apt-get update && apt-get dist-upgrade -y

# We can replace this port with what the user wants
EXPOSE 8080

# Add utils files
ADD https://raw.githubusercontent.com/IBM-Swift/swift-ubuntu-docker/master/utils/run-utils.sh /swift-utils/run-utils.sh
ADD https://raw.githubusercontent.com/IBM-Swift/swift-ubuntu-docker/master/utils/common-utils.sh /swift-utils/common-utils.sh
RUN chmod -R 555 /swift-utils

# Bundle application source & binaries
COPY --from=builder /swift-project /swift-project

CMD [ "sh", "-c", "cd /swift-project && .build-ubuntu/release/KituraInit" ]
