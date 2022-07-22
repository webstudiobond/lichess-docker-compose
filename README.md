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
   # get full path to lichess folder
   lichess_path=$(realpath .)
   
   # run docker image for buil and complile lila
   docker run \
     --name lila \
     --interactive \
     --tty \
     --rm \
     --volume $lichess_path/lila/source:/home/lichess/lila \
     --volume $lichess_path/lila/data/application.conf:/home/lichess/lila/conf/application.conf \
     --volume $lichess_path/cache:/home/lichess/.cache \
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
   # get full path to lichess folder
   lichess_path=$(realpath .)
   
   # run mongo db
   docker run \
     --name lila_db \
     --rm \
     --volume $lichess_path/lila_mongo/db:/data/db \
     --volume $lichess_path/lila_mongo/configdb:/data/configdb \
     --volume $lichess_path/lila/source/bin/mongodb/indexes.js:/host/lila/bin/mongodb/indexes.js \
     --volume $lichess_path/lila-db-seed:/host/lila-db-seed \
     mongo:5.0
```
   
   In new terminal window
   ```
   # create db indices
   docker exec -it lila_db mongo lichess /host/lila/bin/mongodb/indexes.js
   
   # optional seed database
   docker exec -it lila_db mongorestore /host/lila-db-seed/dump
   ```
   Optional [populate your database](https://github.com/lichess-org/lila-db-seed#populate-your-database)
```
# get full path to lichess folder
lichess_path=$(realpath .)

# run docker image for lila-db-seed/spamdb/spamdb.py script.
docker run \
     --name lila-pip3 \
     --interactive \
     --tty \
     --rm \
     --volume $lichess_path/lila-db-seed:/home/lichess/lila-db-seed \
     --user 1000:1000 \
     --workdir /home/lichess/lila-db-seed \
     --link lila_db:db \
     sb/lichess_base

# temporary install python & pip, install pymongo
sudo apt update && sudo apt install python3 python3-pip -y && pip3 install pymongo

# usage help
python3 /home/lichess/lila-db-seed/spamdb/spamdb.py --help

# example. import users 2 normal users and all special users from ./lila-db-seed/spamdb/data/uids.txt
python3 /home/lichess/lila-db-seed/spamdb/spamdb.py \
 --uri mongodb://db:27017/lichess \
 --password strong_password \
 --users 0 \
 --posts 0 \
 --blogs 0 \
 --tours 0 \
 --games 0 \
 --follow 0 \
 --teams 1 \
 --no-timeline

# exit
exit
```
   Stop mongo db. `Ctrl + c` in first terminal window with mongo db.

14. Optional add ["play against the computer"](https://github.com/lichess-org/lila/wiki/Lichess-Development-Onboarding#optional-setup-play-with-the-computer). if you don't need it, comment out the services `lila-fishnet` and `lila-fishnet-client` in `docker-compose.yaml`

```
git clone https://github.com/lichess-org/lila-fishnet.git  ./lila-fishnet
git clone --recursive https://github.com/lichess-org/fishnet.git ./fishnet
chown -R 1000:1000 lila-db-seed /home/chess/lichess/{lila,lila-ws,cache,lila-fishnet,fishnet}
```

15. Optional: [setup Stockfish analysis](https://github.com/lichess-org/lila/wiki/Lichess-Development-Onboarding#optional-setup-stockfish-analysis). if you don't need it, comment out the service `lila-stockfish-analysis` in `docker-compose.yaml`

16. Run lichess docker compose.

`docker compose -f ./docker-compose.yaml up -d`

17. You should also read the [Lichess Development Onboarding guide](https://github.com/ornicar/lila/wiki/Lichess-Development-Onboarding#installation) on the [Lichess GitHub](https://github.com/ornicar/lila/wiki) wiki for additional instructions on seeding the db, gaining admin access, or running suplementary services like fishnet for server analysis or playing vs Stockfish.


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

## Optional
If you do not want to add content from spamdb to the database, like me, but you need [special users](https://github.com/lichess-org/lila-db-seed#special-users). Go to website, create a user. 
If you want to make a user [admin](https://github.com/lichess-org/lila/wiki/Lichess-Development-Onboarding#development), connect to the lichess db with mongo lichess and run

```docker exec -it lila_db mongo lichess db.user4.update({ _id: "your_id" }, {$set: {roles: ["ROLE_SUPER_ADMIN"]}})```

With `your_id` being the username in lowercase.

You can also run docker container [`Mongo Express`](https://github.com/mongo-express/mongo-express) for edit database.
```
docker run -it --rm \
    --name mongo-express \
    --network lichess_default \
    -p 8081:8081 \
    -e ME_CONFIG_OPTIONS_EDITORTHEME="ambiance" \
    -e ME_CONFIG_BASICAUTH_USERNAME="user_for_web_authentication" \
    -e ME_CONFIG_BASICAUTH_PASSWORD="user_password_for_web_authentication" \
    -e ME_CONFIG_MONGODB_URL="mongodb://db:27017/lichess" \
    mongo-express
```

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
