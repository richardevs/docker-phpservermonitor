
# PHPServerMonitor in Docker (Last version 3.3.2)

### Last update : 2019/06/20 . Created repository with version 3.3.2
### Small modification by richardevs for personal use, forked from AustinSaintAubin/docker-phpservermonitor

### PHPServerMonitor

[PHPServerMonitor](http://www.phpservermonitor.org/) is a script that checks whether your websites and servers are up and running. It comes with a web based user interface where you can manage your services and websites, and you can manage users for each server with a mobile number and email address.

### Docker

[Docker](https://www.docker.com/) allows you to package an application with all of its dependencies into a standardized unit for software development.

More information : 

* [What is docker](https://www.docker.com/what-docker)
* [How to Create a Docker Business Case](https://www.brianchristner.io/how-to-create-a-docker-business-case/)

### Information & Build

```
## Build and Deploy PHPServerMonitor with Docker

# Working Dir
mkdir -p "/tmp/php-server-monitor"
cd "/tmp/php-server-monitor"

# Clone Git Repo
git clone https://github.com/phpservermon/phpservermon.git phpservermonitor
cd phpservermonitor/

# Build Docker Image
docker build --no-cache \
  --tag "phpservermonitor:3.4.0" \
  --tag "phpservermonitor:latest" \
  --file Dockerfile .
```


## Option 1 (recommended): Deploy Container Stack w/ Docker-Compose & Random Passwords

```
# Generate Random Passwords
export "MYSQL_ROOT_PASSWORD=$(cat /dev/urandom | tr -dc 'a-zA-Z0-9' | fold -w 32 | head -n 1)" "MYSQL_PASSWORD=$(cat /dev/urandom | tr -dc 'a-zA-Z0-9' | fold -w 32 | head -n 1)"

# Deploy with Docker-Compose
docker-compose up -d
```

## Option 2: Deploy Seperated Container w/ Docker-Compose & Random Passwords

```
# Start Containers w/ Random Passwords
export "MYSQL_ROOT_PASSWORD=$(cat /dev/urandom | tr -dc 'a-zA-Z0-9' | fold -w 32 | head -n 1)" "MYSQL_PASSWORD=$(cat /dev/urandom | tr -dc 'a-zA-Z0-9' | fold -w 32 | head -n 1)"

# phpservermonitor
docker run -d --name phpservermonitor \
  --restart always \
  --link phpservermonitor_mariadb \
  -v /sessions \
  --dns=192.168.3.8 \
  -p 8080:80 \
  -e TIME_ZONE='America/Chicago' \
  -e PSM_REFRESH_RATE_SECONDS=15 \
  -e PSM_AUTO_CONFIGURE=true \
  -e MYSQL_HOST=database \
  -e MYSQL_USER=phpservermonitor \
  -e MYSQL_PASSWORD=${MYSQL_PASSWORD} \
  -e MYSQL_DATABASE=phpservermonitor \
  -e MYSQL_DATABASE_PREFIX=psm_ \
  phpservermonitor:latest

# phpservermonitor_mariadb
docker run -d --name phpservermonitor_mariadb \
  --restart always \
  -e MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD} \
  -e MYSQL_USER=phpservermonitor \
  -e MYSQL_PASSWORD=${MYSQL_PASSWORD} \
  -e MYSQL_DATABASE=phpservermonitor \
  mariadb
```

#### Database Configuration

Configuration should be automatic if the envirmental varable is set `PSM_AUTO_CONFIGURE=true`. Otherwise the configuration window should be prepopulated to the following.

* **Database Host:** database
* **Database Name:** phpservermonitor
* **Database User:** phpservermonitor
* **Data Password:** YOUR_PASSWORD
* **Table Previx:** psm_

-----

## Know Bugs

 - Servers whos status is cheched though "Ping" will fail when the "Update" button is pressed in the UI. This is because the UI runs as user 'www-data', where as the cron refresh task runs as root. You need root permissions to run SOCK_RAW on Unix systems, to the best of my knowledge there is no way around this. https://secure.php.net/manual/en/function.socket-create.php 
