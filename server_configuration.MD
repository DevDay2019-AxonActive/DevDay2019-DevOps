# Server configuration

Background: if your host is completely new, you should follow these steps from the beginning. Otherwise, if the host is available ELK and ready to deploy your implemented service, please ignore and find your step at the end.

## Basic configuration
### Access server via ssh
- `ssh username@host_ip`
- enter password

### Switch to root user to be able use docker
- `sudo su root`

### Set language for host (if it is not English)
- `sudoedit /etc/default/locale`
- paste these content:
```
LANG="en_US.UTF-8"
LANGUAGE="en_US:en"
```

### Create new user account (in case you want to have another account)
- `adduser <username>`
- enter password

### Grant root permission to user (if needed)
- `sudo su root`
- `usermod -aG sudo <username>`

### Install maven
- `sudo apt-get install software-properties-common`
- `sudo apt-add-repository universe`
- `sudo apt-get update`
- `sudo apt-get install maven`

### Install Postgres client (if you want to test Postgres db inside your host)
- `sudo apt-get install postgresql-client`
- `check postgres is running: ps -ef | grep postgres`

### Download and start Postgres docker container
Source: https://medium.com/better-programming/connect-from-local-machine-to-postgresql-docker-container-f785f00461a7
- `docker run -d -p 5432:5432 --name my-postgres -e POSTGRES_PASSWORD=<password> <database name>`

### Create new postgres username and database using psql
- exec docker container to use psql: `docker exec -it <container name> bash`
- switch to postgres username inside docker container: `su - psql`
```
postgres=# create database mydb;
postgres=# create user myuser with encrypted password 'mypass';
postgres=# grant all privileges on database mydb to myuser;
postgres=# \q
```
- grant access to table: `postgres=# grant all privileges on all tables in schema public to <username>`
- test database connection by psql: `psql -d "host=localhost port=5432 dbname=taop user=taop`

### Change Postgres role password
- access docker container using docker exec: `docker exec -it <container_id> bash`
- change to postgres username: `su - postgres`
- open psql command: `psql <database_name> <role_name>`
- `postgres=# \password <role_name>`
- enter your new password
- it done, quit: `postgres=# \q`

### Install Git
- `apt-get update`
- `apt-get install git`
- `git --version`

### Install docker-compose
#### install via apt
- `apt install docker-compose`
#### install via curl
- `apt-get install curl`
- `sudo curl -L "https://github.com/docker/compose/releases/download/1.23.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose`
- `grant docker-compose permission (if you got permission denied): chmod +x /usr/local/bin/docker-compose`

### Pull and run ELK from Jhipster console modules
- `git clone https://github.com/vutran92/jhipster-console.git`
- `cd jhipster-console/`
- run ELK: `docker-compose up -d`

### Deploy backend service using Docker
- clone source or download your JAR file from local
- `cd /supporter/DevDay2019-Performance-Tuning`
- build JAR file from source-code: `mvn clean install -DskipTests`
- build docker image and name it: `docker build --rm --tag devday:latest .`
- run docker image with configuration: `docker run --rm -it -d -v /usr/share/logs/data:/logs -p <external port>:8080 devday:latest`

### Deploy backend service without Docker
- upload JAR file or clone source-code and then build JAR file
- run service by java command: java -jar <location to JAR file>
- or run service by Maven: mvn spring-boot:run

### Check log using Kibana
- go to your Kibana host and using Logstrail, DevTools or Dashboard to show your log. Default Kibana port is `5601`.
