# Docker - Overview
## Intro
Creating application images and running containers from that image provides for a networked application to test
development code. For example connecting NGINX, MySQL, Laravel and REDIS.

## Images and containers
A docker image is like a model and each container an invocation of
that model. List images

```docker image ls```

Image IDs only

```docker image ls -q```

### Run a container
```
    docker run --rm -d -v $(pwd):/application:/var/www/html -p 8080:80 laraveldocker/phpapp
```

Options for build:

```
Build an image from a Dockerfile

Options:
      --add-host list           Add a custom host-to-IP mapping (host:ip)
      --build-arg list          Set build-time variables
      --cache-from strings      Images to consider as cache sources
      --cgroup-parent string    Optional parent cgroup for the container
      --compress                Compress the build context using gzip
      --cpu-period int          Limit the CPU CFS (Completely Fair
                                Scheduler) period
      --cpu-quota int           Limit the CPU CFS (Completely Fair
                                Scheduler) quota
  -c, --cpu-shares int          CPU shares (relative weight)
      --cpuset-cpus string      CPUs in which to allow execution (0-3, 0,1)
      --cpuset-mems string      MEMs in which to allow execution (0-3, 0,1)
      --disable-content-trust   Skip image verification (default true)
  -f, --file string             Name of the Dockerfile (Default is
                                'PATH/Dockerfile')
      --force-rm                Always remove intermediate containers
      --iidfile string          Write the image ID to the file
      --isolation string        Container isolation technology
      --label list              Set metadata for an image
  -m, --memory bytes            Memory limit
      --memory-swap bytes       Swap limit equal to memory plus swap:
                                '-1' to enable unlimited swap
      --network string          Set the networking mode for the RUN
                                instructions during build (default "default")
      --no-cache                Do not use cache when building the image
      --pull                    Always attempt to pull a newer version of
                                the image
  -q, --quiet                   Suppress the build output and print image
                                ID on success
      --rm                      Remove intermediate containers after a
                                successful build (default true)
      --security-opt strings    Security options
      --shm-size bytes          Size of /dev/shm
      --squash                  Squash newly built layers into a single
                                new layer
  -t, --tag list                Name and optionally a tag in the
                                'name:tag' format
      --target string           Set the target build stage to build.
      --ulimit ulimit           Ulimit options (default [])
```

To start containers in the background:  
``` docker-compose up ```   
``` docker-compose down ```   
Check running containers  
``` docker ps ```  
Stop container  
``` docker stop [CONTAINER_ID] ```  
Remove container  
``` docker rm [CONTAINER_ID] ```  
Remove all  
``` docker image rm $(docker image ls -q) ```

## Docker images

### Aim
Create a dev environment with the following containers:
- NGINX
- PHP
- MySQL
- Redis

### Steps
1. Create the Dockerfile
2. use official images for MySQL and REDIS
3. Create a new directory for the app
   1. ``` mkdir -p docker/project/app ```
   2. ``` cd docker/project/app ```
   3. Create Dockerfile  
   ```FROM ubuntu:latest```
   4. ```docker run --rm -it ubuntu:latest bash```
4. Test the container and use package manager to install NGINX   
   ```apt-get update```  
   ```apt-get install nginx -y```
5. Check installation   
   ```which nginx```
6. Check container
   ```docker ps```   
   ```docker diff [CONTAINER_ID]```
7. Commit the running container with changes
   ```docker commit -a "Honky Dury" -m "Installed NGINX" [CONTAINER_ID] [REPOSITORY_NAME:TAG]```  
   
8. ```docker image ls```
   ```
   REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
   mynginx      latest    057a5b43c16a   10 seconds ago   169MB
   ubuntu       latest    27941809078c   5 weeks ago      77.8MB
   ```
9. Run image
   ```docker run --rm -it mynginx:latest bash```
10. Every line in the Dockerfile will create a new container, the final image is the result of the chain of container builds.

## Dockerfiles

Source: [Servers for Hackers](https://serversforhackers.com/c/div-using-dockerfiles)

``` 
FROM ubuntu:22.04

LABEL maintainer="Mack Knave"

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update \
    && apt-get install -y gnupg tzdata \
    && echo "UTC" > /etc/timezone \
    && dpkg-reconfigure -f noninteractive tzdata

RUN apt-get update \
    && apt-get install -y curl zip unzip git supervisor sqlite3 \
       nginx php8.1-fpm php8.1-cli \
       php8.1-pgsql php8.1-sqlite3 php8.1-gd \
       php8.1-curl php8.1-memcached \
       php8.1-imap php8.1-mysql php8.1-mbstring \
       php8.1-xml php8.1-zip php8.1-bcmath php8.1-soap \
       php8.1-intl php8.1-readline php8.1-xdebug \
       php-msgpack php-igbinary \
    && php -r "readfile('http://getcomposer.org/installer');" | php -- --install-dir=/usr/bin/ --filename=composer \
#    && mkdir /run/php \
    && apt-get -y autoremove \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
```
Then build the container  
```docker build -t harddocker/app:latest -f ./Dockerfile .```

``` 
[+] Building 132.5s (7/7) FINISHED                                              
 => [internal] load build definition from Dockerfile                       0.1s
 => => transferring dockerfile: 928B                                       0.0s
 => [internal] load .dockerignore                                          0.0s
 => => transferring context: 2B                                            0.0s
 => [internal] load metadata for docker.io/library/ubuntu:22.04            1.0s
 => [1/3] FROM docker.io/library/ubuntu:22.04@sha256:b6b83d3c331794420340  0.0s
 => CACHED [2/3] RUN apt-get update     && apt-get install -y gnupg tzdat  0.0s
 => [3/3] RUN apt-get update     && apt-get install -y curl zip unzip g  128.1s
 => exporting to image                                                     3.1s
 => => exporting layers                                                    3.1s
 => => writing image sha256:1729210872dce9c41720d96cab53f1f5c215025c72605  0.0s 
 => => naming to docker.io/harddocker/app:latest                           0.0s 
```
The 'context' is relative to file or directory resources referenced within the Dockerfile

```
docker image ls
REPOSITORY       TAG       IMAGE ID       CREATED         SIZE
harddocker/app   latest    1729210872dc   8 minutes ago   353MB
``` 
Build in Docker desktop:

Show containers:   
```docker ps```  
```docker ps -a ```  # To show all

Run NGINX in the foreground: 
1. ```&& echo "daemon off"; >> /etc/nginx/nginx.conf```  
2. ```docker build -t harddocker/app:latest -f ./Dockerfile .```
3. ```docker image ls```
4. ```docker run -d harddocker/app:latest nginx```

## Serving web files
```docker ps -a```  
```docker stop [CONTAINER_ID] && docker rm [CONTAINER_ID]```  

Create NGINX folder for default.conf
1. ```mkdir docker/project/app/nginx```  
2. ```nano docker/project/app/nginx/default```
```
server {
    listen 80 default_server;

    root /var/www/html/public;

    index index.html index.htm index.php;

    server_name _;

    charset utf-8;

    location = /favicon.ico { log_not_found off; access_log off; }
    location = /robots.txt  { log_not_found off; access_log off; }

    location / {
        try_files $uri $uri/ /index.php$is_args$args;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.0-fpm.sock;
    }

    error_page 404 /index.php;
}
```
3. ```ADD nginx/default /etc/nginx/sites-available/default```
4. ```docker build -t harddocker/app:latest -f ./Dockerfile .```
5. ```docker run --rm -d -v $(pwd)/application:/var/www/html/public -p 8080:80 docker.io/harddocker/app:latest nginx```
6. ```curl localhost:8080``` pulls the NGINX server page
7. 502 error on http://localhost:8080/index.php
8. ```docker exec -it [CONTAINER_ID] bash```
9. ```# ps aux```  
```
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.4  0.0   4624  3704 pts/0    Ss+  22:43   0:00 bash
root        12  0.5  0.0   2888   936 pts/1    Ss   22:43   0:00 /bin/sh
root        18  0.0  0.0   7060  1572 pts/1    R+   22:44   0:00 ps aux
```  
Evidently php is not running, and so we must start the php-fpm process. 

## Running multiple processes
We installed supervisord so that we could run multiple processes within this container. 

supervisord is a process monitor and can be configured to for a set of services to monitor.

Create a supervisord.conf 
1. ```mkdir process```  
2. ```nano process/supervisord.conf```   
``` 
[supervisord]
nodaemon=true

[program:nginx]
command=nginx
stdout_logfile=/dev/stdout
stdout_logfile_maxbytes=0
stderr_logfile=/dev/stderr
stderr_logfile_maxbytes=0

[program:php-fpm]
command=php-fpm8.1
stdout_logfile=/dev/stdout
```
4. ```ADD ./process/supervisord.conf /etc/supervisor/conf.d/supervisord.conf``` 
5. ```docker build -t harddocker/app:latest -f ./Dockerfile .```
6. ```docker run --rm -d -v $(pwd)/application:/var/www/html/public -p 8080:80 docker.io/harddocker/app:latest supervisord```
7. ```docker exec -it [CONTAINER_ID] bash```
8. From inside the container: ```ps aux```
9. ```curl localhost:80/index.php```
10. From outside: ```curl localhost:8080/index.php```
11. Browser: ```http://localhost:8080/index.php``` 

Running the command: 
1. Add to docker file ```CMD ["supervisord"]```
2. ```docker build -t harddocker/app:latest -f ./Dockerfile .```
3. ```docker run --rm -d -v $(pwd)/application:/var/www/html/public -p 8080:80 docker.io/harddocker/app:latest```
4. ```docker exec -it [CONTAINER_ID] bash```
5. ```ps aux```
6. ```http://localhost:8080/index.php``` 

Stop container

## Configuring PHP-FPM
Customise PHP

```mkdir ../docker/project/app/php```

1. ```docker run --rm -d -v $(pwd)/application:/var/www/html/public -p 8080:80 docker.io/harddocker/app:latest```
2. ```docker exec -it [CONTAINER_ID] bash```
3. ```cd /etc/php/8.1/fpm```
4. Copy the php-fpm.conf to the docker/project/app folder
   ```docker exec -it [CONTAINER_ID] cat /etc/php/8.1/fpm/php-fpm.conf | tee php/php-fpm.conf```
5. ```ADD ./php/php-fpm.conf /etc/php/8.1/fpm/php-fpm.conf```
6. ```docker build -t harddocker/app:latest -f ./Dockerfile .```
7. ```docker run --rm -d -v $(pwd)/application:/var/www/html/public -p 8080:80 docker.io/harddocker/app:latest```

## Docker logs

```docker logs [CONTAINER_ID] -f```

### NGINX
Symlink stdout and stderr
```
RUN ln -sf /dev/stdout /var/log/nginx/access.log \
    && ln -sf /dev/stderr /var/log/nginx/error.log
```
       
### PHP-FPM
In the php-fpm.conf file set the error log to:   
```error_log=/proc/self/fd/2```

## Entrypoint or CMD
Using Entrypoint to run a setup script every time we generate a container from an image.
1. In directory ```nano project/app/scripts/start-container.sh``` 
2. Remove ```CMD ["supervisord"]```
3. Add this block:  
``` 
ADD ./scripts/start-container.sh /usr/bin/start-container.sh   
RUN chmod +x /usr/bin/start-container.sh  

ENTRYPOINT ["start-container"]
```
4. ```docker exec -it [CONTAINER_ID] bash```
5. ```which env``` get the path `/usr/bin/env`
6. Write this bash script in scripts/start-container.sh . The script will configure composer and 
set permissions. In addition, it permits docker exec commands or starts supervisord.

```
#!/usr/bin/env bash

##
# Ensure /.composer exists and is writable
#
if [ ! -d /.composer ]; then
    mkdir /.composer
fi

chmod -R ugo+rw /.composer

##
# Run a command or start supervisord
#
if [ $# -gt 0 ];then
    # If we passed a command, run it
    exec "$@"
else
    # Otherwise start supervisord
    /usr/bin/supervisord
fi 
```
9. ```docker build -t harddocker/app:latest -f ./Dockerfile .```
10. ```docker run --rm -d -v $(pwd)/application:/var/www/html/public -p 8080:80 docker.io/harddocker/app:latest```
11. This is not working - cannot locate script - so run with ```CMD ["supervisord"]```

## Docker networks
```docker network ls```

### Create a network
``` 
docker network create mainnet 
docker network ls 
```

### Create a MariaDB container 
1. Get MariaDB from [DockerHub](https://hub.docker.com/_/mariadb)   
```
docker run --detach --network mainnet --name mymariadb --env MARIADB_USER=maria --env MARIADB_PASSWORD=secret --env MARIADB_ROOT_PASSWORD=root  mariadb:latest
docker run -it --network mainnet --rm mariadb mysql -hmymariadb -umaria -p 
...
docker image ls
docker network ls
docker network inspect mainnet
```
2. Gain access to container:
``` 
docker run -it --rm mariadb mysql -h <server container IP> -umaria -p
```
OR  
``` 
docker exec -it mymariadb bash
which mariadb
getent hosts mariadb
mariadb -hmymariadb -umaria -p 
```
3. Stop running containers and add the build to the network:    
```docker run --name app --network mainnet --rm -d -v $(pwd)/application:/var/www/html/public -p 8080:80 docker.io/harddocker/app:latest```
4. ```docker ps```
5. ```docker exec -it app bash```
6. ```getent hosts app```
7. ```docker network inspect mainnet```
``` 
[
    {
        "Name": "mainnet",
        "Id": "86ecec42b47e9dbc913ce105d5c3c5a720071342b3e3d44577b5c8063f80b6db",
        "Created": "2022-07-14T10:09:26.714365525Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "2e7e93215f36bb0db1e1296e89d70345237db68b4f65d37d90df3b0a726c526b": {
                "Name": "zen_mendeleev",
                "EndpointID": "62ca5c7bd581f91dc7b9de4421354b57dde4cf7b86e0b68c7f4a40cc088c7925",
                "MacAddress": "02:42:ac:12:00:03",
                "IPv4Address": "172.18.0.3/16",
                "IPv6Address": ""
            },
            "332d4141b6423aaabdd9d2ca223f30324cb1cb9182c76522d98dfcfbb745c0df": {
                "Name": "suspicious_moser",
                "EndpointID": "718c5d2f9fe98f51eb8eafc4fcdc8c164f8a1155f689181d6a8db3e3e79d6f0e",
                "MacAddress": "02:42:ac:12:00:04",
                "IPv4Address": "172.18.0.4/16",
                "IPv6Address": ""
            },
            "5979c44df52b204340fc517b2033cf41b60d1c4ea4e2a5f2ad61c14ea87905d9": {
                "Name": "mymariadb",
                "EndpointID": "a7b9380a6f6a39e75b7cb1ed6c7fe72ab27071c3dcca016fdd38a272b0db0da6",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
```

## Connecting containers

1. Stop all containers: 
```docker ps ```   
```docker stop [CONTAINER_ID]```
2. Remove the application folder 
```rm -Rf application```
3. ```docker run -it --rm -v $(pwd)/application:/var/www/html/public -w /var/www/html docker.io/harddocker/app:latest```
4. Install Laravel 
```composer create-project laravel/laravel application```
5. Map the application directory (Note the port change)
```docker run --name app --network mainnet --rm -d -v $(pwd)/application:/var/www/html/public -p 80:80 docker.io/harddocker/app:latest```
6. ```docker exec -it -w /var/www/html app php -v``` 
7. ```docker exec -it -w /var/www/html app php artisan -v```


## Bootstrapping Laravel
1. Install UI & Auth (Breeze starter package)
``` 
docker exec -it -w /var/www/html app php artisan composer require laravel/breeze --dev
```
Blade
```
docker exec -it -w /var/www/html app php artisan breeze:install
```
Vue (use --ssr switch to compile for SSR)
```
docker exec -it -w /var/www/html app php artisan breeze:install vue
```
React
```
docker exec -it -w /var/www/html app php artisan breeze:install vue
```
Next.js
```
docker exec -it -w /var/www/html app php artisan breeze:install api
```
Run NPM
```
docker exec -it -w /var/www/html app php artisan npm install
docker exec -it -w /var/www/html app php artisan npm run dev
```
3. Install Inertia
4. Update .env with MariaDB details
``` 
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=mymariadb
DB_USERNAME=maria
DB_PASSWORD=secret
```
5. Migrate
   ```docker exec -it -w /var/www/html app php artisan migrate```
6. Inspect the network ```docker network inspect mainnet```
7. Using Vite

## Docker volumes
``` 
docker volume ls
# Kill MySql
docker kill [CONTAINER_ID]
docker volume create mydata
docker run -d --rm --name=mysql -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=testdb -e MYSQL_USER=maria -e MYSQL_PASSWORD=secret --network=mainnet -v mydata:/var/lib/mysql mariadb:10.6 
```
1. Update application/.env
``` 
DB_CONNECTION=mysql
DB_HOST=mysql
DB_PORT=3306
DB_DATABASE=testdb
DB_USERNAME=maria
DB_PASSWORD=secret
```
2. Migrate
```docker exec -it -w /var/www/html/public app php artisan migrate```
3. Check tables
```docker exec -it mysql mysql -umaria -p -e "USE testdb; SHOW TABLES;"```
4. Kill the container
```docker kill [CONTAINER_ID]```
5. ```docker volume ls```
6. Spin out a new instance
```docker run -d --rm --name=mysql -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=testdb -e MYSQL_USER=maria -e MYSQL_PASSWORD=secret --network=mainnet -v mydata:/var/lib/mysql mariadb:10.6 ```
7. All the tables are still persisted in the database after the kill
   ```docker exec -it mysql mysql -umaria -p -e "USE testdb; SHOW TABLES;"```

# Docker Compose
[docker-compose](https://docs.docker.com/compose/) manuals

## Docker Compose intro

## Docker Compose services
1. Check which volumes are stored:
```
docker volume ls
docker kill [CONTIANER_ID]
```
2. Remove volumes
```
docker volume rm $(docker volume ls -q)
```
3. Remove any networks
   ```docker network rm mainnet```
4. Create a docker file
```
version: "3.9"  # optional since v1.27.0
services:
  cache:
    image: redis:alpine
    networks:
      - mainnet
  db:
    image: mariadb:latest
    environment:
      - MYSQL_ROOT_PASSWORD=root
      - MYSQL_DATABASE=testdb
      - MYSQL_USER=maria
      - MYSQL_PASSWORD=secret
    networks:
      - mainnet
networks:
  mainnet:
    driver: bridge
volumes:
  dbdata:
    driver: local
  cachedata:
    driver: local
```
5. Test the docker-compose file
```
docker-compose ps
docker-compose up -d 
docker image ls
docker ps
docker network ls
docker volume ls
```
6. Stop and check
```
docker-compose down
docker ps -a
docker volume ls 
``` 
7. Start volumes again
   ```docker-compose up -d```

## Docker Compose, using volumes
Update the docker-compose.yml
``` 
  db:
    image: mariadb:latest
    environment:
      - MYSQL_ROOT_PASSWORD=root
      - MYSQL_DATABASE=testdb
      - MYSQL_USER=maria
      - MYSQL_PASSWORD=secret
    networks:
      - mainnet
    volumes:
      - dbdata:/var/lib/mysql
```
1. Remove volumes  
   ```docker-compose down && docker volume rm $(docker volume ls -q)```
2. Start
```
docker-compose up -d 
docker volume ls
```
3. Check that all components are built
```
docker-compose down && docker-compose up 
docker network ls && docker volume ls
```

## App services
1. Update the application .env
   ```composer create-project laravel/laravel application```
   application/.env
``` 
DB_CONNECTION=mysql
DB_HOST=db
DB_PORT=3306
DB_DATABASE=homestead
DB_USERNAME=maria
DB_PASSWORD=secret

BROADCAST_DRIVER=log
CACHE_DRIVER=redis
FILESYSTEM_DISK=local
QUEUE_CONNECTION=sync
SESSION_DRIVER=redis
SESSION_LIFETIME=120

REDIS_HOST=cache
REDIS_PASSWORD=null
REDIS_PORT=6379
```
2. Update the docker-compose.yml
``` 
services:
  app:
    build: 
      context: ./app
      dockerfile: ./Dockerfile
    image: harddocker/app:latest
    networks:
      - mainnet
    volumes:
      - ./app/application:/var/www/html
    ports:
      - 80:80
``` 

3. Add predis to the build:
   ```docker-compose exec app bash -c "cd /var/www/html && composer require predis/predis"```
4. Migrate
   ```docker-compose exec app bash -c "cd /var/www/html && php artisan migrate"```
5. Check tables
## Working directory
1. Update app in docker-compose.yml
```
services:
  app:
    ...
    ports:
      - 80:80
    working_dir: /var/www/html  
```
2. ```docker-compose restart```
3. ```docker-compose exec app pwd```
4. ```docker-compose run app pwd```
5. ```docker-compose exec app bash -c "cd /var/www/html && pwd"```
6. Up down turn around
```
docker-compose down && docker-compose up -d 
docker-compose exec app pwd
```

## Variables
Variables are called using ${VAR_NAME}
1. Create .env for docker-compose.yml
```
APP_PORT=8080
DB_PORT=33060
```
2. Add to docker-compose.yml
```
  app:
    ( ... )
    ports:
      - ${APP_PORT}:80
  db:
    ( ... )
    ports:
      - ${DB_PORT}:3306
```
2. ```docker-compose ps```
3. ```docker-compose down && docker-compose up -d```
## Adding nodejs as a service
1. Update docker-compose.yml
``` 
  node:
    build:
      context: ./node
      dockerfile: Dockerfile
    image: harddocker/node:latest
    networks:
     - mainnet
    volumes:
     - .:/opt
    working_dir: /opt
    command: echo hello world
```
2. ```mkdir node && cd node && touch Dockerfile```
3. Add to ./node/Dockerfile
``` 
FROM node:latest

LABEL maintainer="Mack Knave"

RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - \
    && echo "deb http://dl.yarnpkg.com/debian/ stable main" > /etc/apt/sources.list.d/yarn.list \
    && apt-get update \
    && apt-get install -y git yarn \
    && apt-get -y autoremove \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* 
```
4. ```docker-compose down && docker-compose up -d```
5. ```docker-compose ps```
6. ```docker-compose run --rm node yarn install```
## Dev workflow intro

## Workflow