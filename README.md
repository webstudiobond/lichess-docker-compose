# Lichess Docker Compose
Docker compose setup for Lichess development. Based on [`benediktwerner`](https://github.com/benediktwerner/lichess-docker) and [`phorcys420`](https://github.com/phorcys420/lichess-docker) lichess-docker.

## Requirements
- [`docker`](https://docs.docker.com/engine/install/) and [`docker compose`](https://docs.docker.com/compose/install/)
- time, and preferably a coffee machine (or actually, weed)

## Description
- `lichess_base`       : A docker image, based on [`debian`](https://hub.docker.com/_/debian)`:stable-slim` that packages Scala, Java & NodeJS.
- `lila`               : An example `application.conf` config for [lila](https://github.com/ornicar/lila) server.
- `lila-ws`            : An example `ws.conf` config for [lila-ws](https://github.com/ornicar/lila-ws) server.
- `nginx_examples`     : An example `nginx.conf` for `chess.exaple.com`.
- `docker-compose.yaml`: `redis`, `mongo`, `lila` and `lila-ws` services.

## Usage

**Note: One may be tempted to replace the `mongodb://db` and `redis://redis` URLs with whatever their mind comes up with, but this is normal!
Linked docker containers will automatically have hostnames.**

**Note: If you're on windows (non-WSL), make sure your `git` config is using Unix line endings (LF) and not Windows ones (CLRF).**

**Note: For example, the working folder will be `lichess`. Don't forget to change the path `./lichess`, shown in the examples below.**


1. Navigate to the folder, where the project will be.
2. Clone [this](https://github.com/seb81/lichess-docker-compose) repo to `./lichess` and `cd` into them.

   `git clone https://github.com/seb81/lichess-docker-compose ./lichess && cd ./lichess`
3. Create a cache folder.

   `mkdir -p ./cache`
4. Create a source folder for `lila` and `lila-ws`.

   `mkdir -p ./{lila,lila-ws}/source`
5. Clone [lila](https://github.com/lichess-org/lila) repo to `./lila/source`.

   `git clone --recursive https://github.com/lichess-org/lila.git ./lila/source`
6. Clone [lila-ws](https://github.com/lichess-org/lila-ws) repo to `./lila-ws/source`.

   `git clone https://github.com/lichess-org/lila-ws.git ./lila-ws/source`
7. Parts of the site (like puzzles) require some minimal content to function. 
   Clone [lila-db-seed](https://github.com/lichess-org/lila-db-seed) repo.

   `git clone https://github.com/lichess-org/lila-db-seed.git`
8. Chown lila, lila-ws and cache.

   `chown -R 1000:1000 ./{lila,lila-ws,cache}`
9. Build the `lichess_base` image.

   `docker build --tag sb/lichess_base ./lichess_base/`
10. Edit the example configuration file located at `./lila/data/application.conf`.

    `nano ./lila/data/application.conf`
11. Edit the CSRF origin in the example config file located at `./lila-ws/data/ws.conf`.

    `nano ./lila-ws/data/ws.conf`
12. Run docker image `lichess_base` for setup [lila](https://github.com/lichess-org/lila/wiki/Lichess-Development-Onboarding).
```
   docker run \
     --name lila \
     --interactive \
     --tty \
     --rm \
     --volume ./lila/source:/home/lichess/lila \
     --volume ./lila/data/application.conf:/home/lichess/lila/conf/application.conf \
     --volume ./cache:/home/lichess/.cache \
     --user 1000:1000 \
     --workdir /home/lichess/lila \
     sb/lichess_base
   
   # Build the CSS and JS
   ./ui/build
   
   # Start the SBT console
   ./lila compile
   
   # exit
   exit
   ```
13. Before the first run, you should create db indices. And (optional) [seed database](https://github.com/lichess-org/lila/wiki/Lichess-Development-Onboarding#optional-seed-database).
```
   docker run \
     --name lila_db \
     --interactive \
     --tty \
     --rm \
     --volume ./lila_mongo/db:/data/db \
     --volume ./lila_mongo/configdb:/data/configdb \
     --volume ./lila/source/bin/mongodb/indexes.js:/host/lila/bin/mongodb/indexes.js \
     --volume ./lila-db-seed:/host/lila-db-seed \
     mongo:5.0
   
   # create db indices
   mongo lichess /host/lila/bin/mongodb/indexes.js
   
   # optional seed database
   mongorestore /host/lila-db-seed/dump
   
   # exit
   exit
   ```
14. Run lichess docker compose.

`docker compose -f ./docker-compose.yaml up -d`

15. You should also read the [Lichess Development Onboarding guide](https://github.com/ornicar/lila/wiki/Lichess-Development-Onboarding#installation) on the [Lichess GitHub](https://github.com/ornicar/lila/wiki) wiki for additional instructions on seeding the db, gaining admin access, or running suplementary services like fishnet for server analysis or playing vs Stockfish.


## Domain structure
```
chess.example.com	1	IN	A       123.123.123.123
ws.chess.example.com. 1	IN	CNAME   chess.meetings.pp.ua.
```

### Nginx [config](https://github.com/seb81/lichess-docker-compose/tree/master/nginx_examples) description:

`chess.example.com` being the main domain.

`ws.chess.example.com` being the websocket server for `chess.example.com`.

**Note: Don't forget to restart nginx after editing the config for the domain**

```nginx -t && service nginx restart```

## Useful commands

* Stop and remove all the Compose containers: `docker compose down`
* Start all the Compose containers: `docker compose up -d`


* View container logs: `docker logs lila`
* Stop the Docker container: `docker stop lila`
* Open a shell in the running container: `docker exec -it lila bash`
* Attach to the Docker container (main process): `docker attach lila`
* Open a shell as root in the running container: `docker exec -u 0 -it lila bash`

In the above commands, `lila` is replaceable by `lila_redis`, `lila_db`, `lila-ws` and any container name in that manner.

## TODO
* Add fishnet.
* Add "play against the computer"
