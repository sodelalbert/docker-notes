# Docker Notes

## Dockerfile

Syntax explained

`WORKDIR` -  
`COPY` -  
`RUN` -  
`CMD` -  
`EXPOSE` -

## Build image

Build docker image based on `Dockerfile` specified in the project.

```bash
docker build -t getting-started .
```

`-t` - option specifies human readable name for the container  
`.` - place where we should look for `Dockerfile` in the project

Execute following command to determine what images are built in your system.

```bash
docker image list
```

### Best Practices for building container

- Debugging size of containers

  ```bash
  docker image history getting-started
  ```

- Building docker file that only when change happens container layers are recreated.

  ```Dockerfile
  FROM node:lts-alpine
  WORKDIR /app
  COPY . .
  RUN yarn install --production
  CMD ["node", "src/index.js"]
  ```

  By copying first dependencies list from `package.json` yarn created dependencies when change is done to the file, not all of the time.

  ```Dockerfile
  FROM node:lts-alpine
  WORKDIR /app
  COPY package.json yarn.lock ./
  RUN yarn install --production
  COPY . .
  CMD ["node", "src/index.js"]
  ```

- Multi-stage build - are used to separate building from runtime.

  In this example, you use one stage (called build) to perform the actual Java build using Maven. In the second stage (starting at FROM tomcat), you copy in files from the build stage. The final image is only the last stage being created, which can be overridden using the --target flag.

  ```Dockerfile
  FROM maven AS build
  WORKDIR /app
  COPY . .
  RUN mvn package

  FROM tomcat
  COPY --from=build /app/target/file.war /usr/local/tomcat/webapps
  ```

- Docker image analysis

  Github project: [drive](https://github.com/wagoodman/dive)

## Running container

Note that changes introduced to running container **instance** container remain saved when you stop container. This applies to packages installed and configuration made after initial container setup.

```bash
docker run -d -p 127.0.0.1:3000:3000 getting-started
```

`-d` - Detached, run container in background  
`-p` - Publish port mapping

List containers

```bash
docker container list
```

List all containers and validate their execution status.

```bash
docker ps --all
```

Stop and remove running container

```bash
docker rm -f  [CONTAINER_ID]
```

## Named Volumes

Volumes provide the ability to connect specific filesystem paths of the container back to the host machine.

If you mount a directory in the container, changes in that directory are also seen on the host machine. If you mount that same directory across container restarts, you'd see the same files.

```bash
docker volume create todo-db
```

Run docker container and add some toto items.

```bash
docker run -dp 127.0.0.1:3000:3000 --mount type=volume,src=todo-db,target=/etc/todos getting-started
```

Remove it and run new instance again - data is persistent!

Digging into volumes infomation

```bash
docker volume inspect todo-db
```

## Bind Mounts

A bind mount is another type of mount, which lets you share a directory from the host's filesystem into the container.

This is often used to mount source code to the container. Might be useful for development purposes.

```bash
docker run -it --mount type=bind,src="$(pwd)",target=/src ubuntu bash
```

## Development container

Development container is using bind mounts to state application - nodemon is watching for changes is JS file and restarts the container.

```bash
docker run -dp 127.0.0.1:3000:3000 \
    -w /app --mount type=bind,src="$(pwd)",target=/app \
    node:18-alpine \
    sh -c "yarn install && yarn run dev"
```

`-dp 127.0.0.1:3000:3000` - detached mode with port binding  
`-w /app` - sets the "working directory" or the current directory that the command will run from  
`--mount type=bind,src="$(pwd)",target=/app` - mount current directory to working directory.

Watching logs from container run

```bash
docker logs [CONTAINER_ID]
```

## Multiple containers

Containers, by default, run in isolation and don't know anything about other processes or containers on the same machine.If you place the two containers on the same network, they can talk to each other.

```bash
docker network create todo-app
```

List all configured docker networks

```bash
docker network ls
```

TODO: What is the meaning of preconfigured networks? (bridge, host, none)

Create MySQL container

```bash
docker run -d \
    --network todo-app --network-alias mysql \
    -v todo-mysql-data:/var/lib/mysql \
    -e MYSQL_ROOT_PASSWORD=secret \
    -e MYSQL_DATABASE=todos \
    mysql:8.0
```

`-v todo-mysql-data:/var/lib/mysql` - Docker automatically creates volume do MySQL data when the command is triggered

`--network-alias mysql` - this is mysql container DHCP hostname

To confirm that database is up and running start interactive terminal

```bash
docker exec -it <mysql-container-id> mysql -u root -p
```

Network debuging container [nicolaka\netshoot](https://github.com/nicolaka/netshoot?tab=readme-ov-file) act like swiss army knife to determine networking issues within containerized application. Run it interactively attaching it to configured network.

```bash
docker run -it --network todo-app nicolaka/netshoot
```

Connect application to database. Use logs command to verify that connection was successful.

```bash
docker run -dp 127.0.0.1:3000:3000 \
  -w /app -v "$(pwd):/app" \
  --network todo-app \
  -e MYSQL_HOST=mysql \
  -e MYSQL_USER=root \
  -e MYSQL_PASSWORD=secret \
  -e MYSQL_DB=todos \
  node:18-alpine \
  sh -c "yarn install && yarn run dev"
```

Add some todo items in Web interface and verify database contents.

```bash
docker exec -it <mysql-container-id> mysql -p todos
```

Execute in mysql database

```
select * from todo_items;
```

## Docker Compose

`docker compose up -d`

`docker compose logs -f`

Teardown of all containers

`docker compose down` - Removes containers and Networks
`docker compose down --volumes` - Remove all volumes in addition to above.

## Docker Compose with Dockerfiles

Project structure

```
project/
│
├── service1/
│   ├── Dockerfile
│   └── app.py
│
├── service2/
│   ├── Dockerfile
│   └── main.js
│
└── docker-compose.yml
```

`docker-compose.yaml`

```yaml
version: "3.9"
services:
  service1:
    build:
      context: ./service1
      dockerfile: Dockerfile
    ports:
      - "5000:5000"

  service2:
    build:
      context: ./service2
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
```

## Cleanup Docker

Reemove all:

- Volumes
- Networks
- Containers (including running ones)

```bash
docker volume rm $(docker volume ls -q);
docker network rm $(docker network ls -q);
docker rm -f $(docker ps -aq)
```

Images are ok, could be cashed for faster runs.
