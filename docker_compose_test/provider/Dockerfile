# Use a base image suitable for your provider
FROM ubuntu:22.04

# Install dependencies for ZeroMQ and the provider (could be a simple Python app, for example)
RUN apt-get update && apt-get install -y \
    build-essential \
    cmake \
    libzmq3-dev \
    python3 \
    python3-pip \
    iputils-ping \
    && apt-get clean

# Copy your provider source code
# COPY . /app
WORKDIR /app
CMD ["echo", "Hello, World!"]

# Command to run your provider (example using Python ZeroMQ)
# CMD ["python3", "provider.py"]