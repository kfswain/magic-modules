# Stage 1: Building Go dependencies
FROM golang:1.19-bullseye AS builder

# Set working directory
WORKDIR /app

# Download go.mod and go.sum from your repository
RUN curl -o go.mod "https://raw.githubusercontent.com/GoogleCloudPlatform/magic-modules/main/tpgtools/go.mod"
RUN curl -o go.sum "https://raw.githubusercontent.com/GoogleCloudPlatform/magic-modules/main/tpgtools/go.sum"

# Install the go dependencies
RUN go mod download

# Stage 2: Creating the final imag
FROM ruby:3.1-bullseye

# golang
COPY --from=golang:1.19-bullseye /usr/local/go /usr/local/go
ENV GOPATH /go
ENV PATH /usr/local/go/bin:$PATH
ENV PATH $GOPATH/bin:$PATH
RUN mkdir -p "$GOPATH/src" "$GOPATH/bin" && chmod -R 1777 "$GOPATH"
WORKDIR $GOPATH

# terraform binary used by tfv/tgc
COPY --from=hashicorp/terraform:1.4.2 /bin/terraform /bin/terraform

SHELL ["/bin/bash", "-c"]

ENV GO111MODULE "on"
ENV LC_ALL=C.UTF-8
ENV LANG=C.UTF-8

RUN apt-get update && apt-get install -y --no-install-recommends git openssh-client apt-transport-https ca-certificates curl netbase wget gcc make jq libjq1

RUN git config --global user.name "Modular Magician"
RUN git config --global user.email "magic-modules@google.com"

# Set up Github SSH cloning.
RUN ssh-keyscan github.com >> /known_hosts
RUN echo "UserKnownHostsFile /known_hosts" >> /etc/ssh/ssh_config

RUN go install golang.org/x/tools/cmd/goimports@d088b475e3360caabc032aaee1dc66351d4e729a
RUN go install github.com/github/hub@v2.11.2+incompatible

ADD "https://raw.githubusercontent.com/GoogleCloudPlatform/magic-modules/main/mmv1/Gemfile" Gemfile
ADD "https://raw.githubusercontent.com/GoogleCloudPlatform/magic-modules/main/mmv1/Gemfile.lock" Gemfile.lock
RUN bundle install
RUN rm Gemfile Gemfile.lock

# Copy Go dependencies from builder stage
COPY --from=builder /go/pkg /go/pkg

ADD generate_downstream.sh /generate_downstream.sh

ENTRYPOINT ["/generate_downstream.sh"]
