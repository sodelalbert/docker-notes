# Use a C++ base image (for example, Ubuntu)
FROM ubuntu:22.04

# Install necessary dependencies for ZeroMQ and your C++ application
RUN apt-get update && apt-get install -y \
    python3\
    iputils-ping

# Copy your consumer C++ source code
# COPY . /app
WORKDIR /app
COPY hello.py /app

# Build your C++ application (example using cmake)
# RUN cmake . && make

# Command to run your consumer application (replace with actual executable)
# CMD ["./consumer"]