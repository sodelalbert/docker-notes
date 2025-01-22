# Docker

docker image list
docker container list
docker network list
docker volume list

docker build -t my-python-app .

docker run -d -p 4000:80 --name [container_name] [image_name]

# Cleanup

docker rm -f $(docker ps -aq); docker system prune -af --volumes



# Docker Compose

`--build` is needed if we made changes to underlying `Dockerfile`

docker-compose up --build -d
docker-compose down --volumes --remove-orphans

docker ps

docker exec -it [ID] bash
docker exec -it provider bash
docker exec -it consumer bash

# Docker networking debuggingF

docker run -it --network [ID] nicolaka/netshoot
dig hostname

# Compose file examples

## Persistency

Bind mount - share folder between host and container.

```yaml
services:
  app:
    image: python:3.9
    container_name: python_app
    volumes:
      - ./app:/app # Bind mount from host './app' to '/app' in the container
    working_dir: /app
    command: python app.py # Command to run your Python app
    ports:
      - "5000:5000"
```

Named mount - `/data` directory will be mounted as persistent file location volume.

```yaml
services:
  redis:
    image: redis:latest
    container_name: redis_container
    volumes:
      - redis_data:/data # Named volume for Redis data
    ports:
      - "6379:6379" # Expose Redis on port 6379

volumes:
  redis_data: # Define the named volume
```
