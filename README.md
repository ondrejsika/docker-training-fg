# Docker Training FG

    2019 Ondrej Sika <ondrej@ondrejsika.com>

Star, fork, create issues & pull requests!

## Usesful Docker Commands

- `docker system prune` - prune unnecessary data from docker - layers, containers, volumes, ...
- `docker system df` - docker disk usage 
- `docker ps -s` - show sizes of containers 


## Docker & Docker Compose Examples

### Startup order

- Docker use `depends_on` when resolve startup order
- Docker doesnt wait to application is being ready - Kubernetes handles this
- Need some workaround
  - __wait-for-it.sh__ - <https://github.com/vishnubob/wait-for-it>

Example: [docker-training-examples/startup-order](https://github.com/ondrejsika/docker-training-examples/tree/master/startup-order)


### Variables

#### Varible Substitution

Run `TAG=2 PORT=80 docker-compose up -d`

```
version: '3.7'
services:
  hello:
    image: ondrejsika/go-hello-world:${TAG:-latest}
    ports:
      - ${PORT:-80}:80
```

### Multiple Composes

See [multiple-composes](https://github.com/ondrejsika/docker-training-examples/tree/master/multiple-composes) example


### Docker Compose CLI Variables

[Reference](https://docs.docker.com/compose/reference/envvars/)

#### Docker Compose

- `COMPOSE_PROJECT_NAME` - Specify prefix for compose containers (same as `docker-compose -p <project name>`)
- `COMPOSE_FILE` - Specify one or multiple compose files separated by `COMPOSE_PATH_SEPARATOR`. (`COMPOSE_FILE=c1.yml:c2.yml docker-compose` is same as `docker-compose -f c1.yml -f c2.yml`)
- `COMPOSE_PATH_SEPARATOR` - default `:` on Unix, `;` on Windows

#### Docker Compose & Docker (cli)

`DOCKER_HOST`, `DOCKER_TLS_VERIFY` and `DOCKER_CERT_PATH` - configures connection to Docker daemon.

See [the docs](https://docs.docker.com/compose/reference/envvars/#docker_host)

### .env file

`.env` file can store all environment variables (for docker containers, for docker compose substitution and docker compose cli) in one place

I recommend you put the `.env` into you `.gitignore`

Try with [variable substitution example](https://github.com/ondrejsika/docker-training-fg#varible-substitution)

## Docker Images

### Image Names

- `debian:9` - __official__ Debian image maintained by Docker on Docker Hub - <https://hub.docker.com/_/debian>
- `ondrejsika/debian:9` - Debian image in user namespace on Docker Hub
- `reg.istry.cz/debian:9` - Debian image on self hosted Docker registry


### Official Images

- OS - eg.: `debian`, `centos`, ...
- Languages - eg.: `python`, `golang`, ...
- Technologies - `nginx`, `postgres`, ...

### Building Own Images

Docker file reference - <https://docs.docker.com/engine/reference/builder/>

Examples in [docker-training-examples](https://github.com/ondrejsika/docker-training-examples)

- [simple image](https://github.com/ondrejsika/docker-training-examples/tree/master/simple-image)
- [multistage image](https://github.com/ondrejsika/docker-training-examples/tree/master/multistage-image)
- [build args](https://github.com/ondrejsika/docker-training-examples/tree/master/build-args)

#### BuildKit

[Docs](https://docs.docker.com/develop/develop-images/build_enhancements/)

Use BuildKit

```
DOCKER_BUILDKIT=1 docker build .
```

Or in config `/etc/docker/daemon.json`:

```
{ "features": { "buildkit": true } }
```

##### Cache (In Docker Build)

Cache

```
# syntax = tonistiigi/dockerfile:runmount20180618
RUN --mount=type=cache,dst=/cache,readonly=false ...
```

Mount host path

```
# syntax = tonistiigi/dockerfile:runmount20180618
RUN --mount=src=/data,dst=/mnt/data,readonly=false ...
```


## Docker Registry

- Gitlab (Open Source version)
- JFrog Artifactory

Gitlab Premium (19 USD/user/month) supports 


### Run registry locally (dev purposes)

Create SSL certs if needed (or provide them for example using Let's Encrypt). Then run this:

```
docker run -d \
  --restart=always \
  --name registry \
  -u `id -u`:`id -g` \
  -v `pwd`/certs:/certs \
  -v `pwd`/registry-data:/var/lib/registry \
  -e REGISTRY_HTTP_ADDR=0.0.0.0:443 \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/fullchain.pem \
  -e REGISTRY_HTTP_TLS_KEY=/certs/privkey.pem \
  -p 443:443 \
  registry:2
```

### Pruning Docker Registry

Use [docker-distribution-pruner](https://gitlab.com/gitlab-org/docker-distribution-pruner) from Gitlab

Example usage for Gitlab Docker Registry

```
EXPERIMENTAL=true docker-distribution-pruner -config=/var/opt/gitlab/registry/config.yml
```

## Volumes

[Official Docs](https://docs.docker.com/storage/volumes/)

- Minio - Open Source S3 - <https://minio.io/>
- Ceph - Distributed storage - <https://ceph.com/>
- Plugin Example (SSHFS) - <https://github.com/vieux/docker-volume-sshfs>


### Mounts (host FS into container)

- Linux - native support
- Winows
  - <https://medium.com/@hudsonmendes/docker-have-a-ubuntu-development-machine-within-seconds-from-windows-or-mac-fd2f30a338e4>
- OSX 
  - <https://docs.docker.com/docker-for-mac/osxfs-caching/>
  - <https://docs.docker.com/storage/bind-mounts/#configure-mount-consistency-for-macos>
  - <https://medium.com/devcupboard/dockerizing-development-environment-in-mac-performance-debugging-dac16aa4524>


### Access container FS

```
docker run --name nginx -d -p 8000:80 nginx
```

#### docker exec

```
docker exec -w /usr/share/nginx/html nginx cat index.html
docker exec -ti -w /usr/share/nginx/html nginx bash
docker exec -w /usr/share/nginx/html nginx sh -c 'echo "<h1>Hello from exec</h1>" > index.html'
docker run --link nginx ondrejsika/curl -s nginx
```

#### docker cp

[Docs](https://docs.docker.com/engine/reference/commandline/cp/)

##### Copy from conteiner

```
docker cp nginx:/usr/share/nginx/html/index.html .
cat index.html
```

##### Copy to container

```
echo "<h1>Hello from cp</h1>" > index.html
docker cp index.html nginx:/usr/share/nginx/html/
docker run --link nginx ondrejsika/curl -s nginx
```

### Backup & Restore

[Docs](https://docs.docker.com/storage/volumes/#backup-restore-or-migrate-data-volumes)

#### Backup Volumes

[Docs](https://docs.docker.com/storage/volumes/#backup-a-container)

You have a Redis with some data in `/data` volume:

```
docker run --name redis -d redis
docker exec redis redis-cli set hello 'hello world'
docker stop redis

# Backup
docker run --rm --volumes-from redis -v $(pwd):/backup ubuntu tar cvf /backup/backup.tar /data
```

#### Restore Volumes

[Docs](https://docs.docker.com/storage/volumes/#restore-container-from-backup)

Run new Redis

```
docker create --name redis2 redis

# Restore data
docker run --rm --volumes-from redis2 -v $(pwd):/backup ubuntu bash -c "cd /data && tar xvf /backup/backup.tar --strip 1"

docker start redis2

# Check it
docker exec redis2 redis-cli get hello

# Clean up
docker rm -f redis redis2
docker run --rm -v $(pwd):/backup ubuntu rm /backup/backup.tar
```

### Backup, Restore & Transfer

My update of "official" backup/restore info

#### Backup Volumes to Image


```
docker run --name redis -d redis
docker exec redis redis-cli set hello 'hello world'
docker stop redis

# Backup
docker run --name backup --volumes-from redis ubuntu tar cvf /backup.tar /data

# Commit
docker commit backup backup-image

# Remove container
docker rm backup
```

#### Restore Volumes form Image

```
docker create --name redis2 redis

# Restore data
docker run --rm --volumes-from redis2 backup-image bash -c "cd /data && tar xvf /backup.tar --strip 1"

docker start redis2

# Check it
docker exec redis2 redis-cli get hello

# Clean up
docker rm -f redis redis2
```

### Database Images with Data

#### Data loaded on conteiner init

```
docker build -t postgres-data-init examples/postgres-data-init
docker run --name pg -d postgres-data-init
sleep 4
docker exec -u postgres pg psql -c "select hello from hello;"
docker rm -f pg
```

#### Data saved in image (data are part of image)

```
docker build -t postgres-data-image--base examples/postgres-data-image
docker run --name pg -d postgres-data-image--base
sleep 4
docker exec -i -u postgres pg psql < examples/postgres-data-image/init.sql
docker stop pg
docker commit pg postgres-data-image
docker rm pg
docker run --name pg -d postgres-data-image
sleep 4
docker exec -u postgres pg psql -c "select hello from hello;"
docker rm -f pg
```

## Versioning

Image has `<repository>:<tag>` where repository is name of your image and tag is version

Example of tags:

- `v1.0.0` - Git tag using [Semantic Versioning](https://semver.org/)
- `master` - Branch name - for dev deployments


## Traefik

Cloud native proxy

<https://traefik.io>

- SSL using Let's Encrypt
- Hot reloads (watching Docker socket)
- Docker / Kubernetes support

Examples:

- Traefik with Let's Encrypt SSL (web + DNS challenge) - <https://github.com/ondrejsika/traefik-le>
- Traefik with external SSL - <https://github.com/ondrejsika/traefik-ssl>


## Gitlab CI

[slides](https://sika.link/gitlab-ci),
[demo gitlab](https://gitlab-demo.xsika.cz)


### Why Gitlab CI

- Docker First - Great Docker & Kubernetes support
- CI Jobs run in Docker
- Environments - Manage deployments running by CI

### Setup Gitlab Runner

Docker from Docker Gitlab Runner - <https://github.com/ondrejsika/gitlab-ci-runner>


## Sources

- Slides for my Docker Training - <https://sika.link/docker>
- Docker Training Examples (repository) - <https://github.com/ondrejsika/docker-training-examples>
- Slides for Gitlab CI Training - <https://sika.link/gitlab-ci>


## Want more

- Kubernetes Courses - <https://skoleni-kubernetes.cz>
  - Kuberentes Training Examles - <https://github.com/ondrejsika/kubernetes-training-example>
  - How to Install Kubernetes on Bare Metal (or VPS) - <https://github.com/ondrejsika/kubernetes-install-bare-metal>
- Gitlab CI - <https://gitlab-ci.cz>

Pripravuji

- Ansible
- Terraform
