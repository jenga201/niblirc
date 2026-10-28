# Intro
This is the ooinuza project for #nibl

# Setup
Install Mysql Docker Container

Compile and Run Ooinuza Docker Container

## Required Software
	Docker

## Database
#### Create mysql container
```
docker network create -d bridge my-bridge
docker pull mysql:8
docker run --network my-bridge --name niblmysql -e MYSQL_ROOT_PASSWORD={somerootpassword} -d mysql
docker exec -i niblmysql mysql -uroot -p{somerootpassword} < db/ooinuza.sql
```

#### Create mysql user
```
docker exec -it niblmysql bash
mysql -u root -p{somerootpassword}
CREATE USER 'ooinuza'@'172.18.0.%' IDENTIFIED BY 'niblempire';
GRANT ALL PRIVILEGES ON 'ooinuza'.* TO 'ooinuza'@'172.18.0.%';
FLUSH PRIVILEGES;
```
## Run with Docker Compose
Start the `niblmysql` stack first (see the `niblmysql` repo), then:
```
cp .env.example .env   # set PUBLIC_DCC_IP
docker compose up -d --build
```
`config/` and `botlists/` are bind-mounted from the project directory (relative to the compose file) to `/config` and `/data/botlists`. Ports 46571-46573 are published for DCC.

## Tests
Unit tests (JUnit 5) run inside the Docker build stage and gate the image: if a test fails, the build fails and no new image is produced.
```
docker compose build    # runs mvn -B clean package -> surefire
```
The tests are non-destructive: they run against an in-memory H2 database (MySQL compatibility mode) seeded from `src/test/resources/schema.sql` (derived from the canonical `niblmysql` schema) with fake bot/pack data - they never touch the production MySQL database or IRC.

Restarting a container does NOT re-run the tests; they only run when the image is rebuilt (i.e. after `pom.xml` or `src/` changes):
```
docker compose restart  # reuses the existing image, no tests
docker compose up -d    # reuses the existing image unless --build is given
```

## Run App (manual)
The public IP is needed to enable dcc chat

Only one range of ports is allowed for dcc chat and must match entries in config/config.txt (todo fix this)

```
docker run -d \
--net='my-bridge' \
--name=niblirc \
--add-host public_dcc_ip:{publicIp} \
-p 46571-46573:46571-46573 \
-v "$(pwd)/config":/config:rw \
-v "$(pwd)/botlists":/data/botlists:rw \
ooinuza
```