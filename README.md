# Nginx, Tomcat, MySQL Example

This repository contains a simple example of how to run [Tomcat](http://tomcat.apache.org/), [mysql](https://www.mysql.com/), and [nginx](https://www.nginx.com/) on [docker](https://www.docker.com/)

## Installation

### Option 1: Running locally with “Docker for Desktop” docker-compose

> 💡 Recommended if you're planning to develop the application.

1. Install tools to run a Docker locally:

   - [Docker for Desktop (Mac/Windows)](https://docs.docker.com/docker-for-mac/install/): It provides Kubernetes support as well ([noted here](https://docs.docker.com/docker-for-mac/kubernetes/)). 
   - [docker-compose](https://docs.docker.com/compose/install/). If you are using brew, you can run the following: `brew install docker-compose`

2. Confirm all the software is installed:

```bash
> docker version
Client: Docker Engine - Community
 Version:           19.03.12
 API version:       1.40
 Go version:        go1.13.10
 Git commit:        48a66213fe
 Built:             Mon Jun 22 15:41:33 2020
 OS/Arch:           darwin/amd64
 Experimental:      false

Server: Docker Engine - Community
 Engine:
  Version:          19.03.12
  API version:      1.40 (minimum version 1.12)
  Go version:       go1.13.10
  Git commit:       48a66213fe
  Built:            Mon Jun 22 15:49:27 2020
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          v1.2.13
  GitCommit:        7ad184331fa3e55e52b890ea95e65ba581ae3429
 runc:
  Version:          1.0.0-rc10
  GitCommit:        dc9208a3303feef5b3839f4323d9beb36df0a9dd
 docker-init:
  Version:          0.18.0
  GitCommit:        fec3683
> docker-compose version
docker-compose version 1.26.2, build eefe0d31
docker-py version: 4.2.2
CPython version: 3.7.7
OpenSSL version: OpenSSL 1.1.1g  21 Apr 2020
```

### Option 2: Running on Google Kubernetes Engine (GKE) TODO

> 💡  Recommended for demos and making it available publicly.

1. Install tools specified in the previous section (Docker, kubectl)

2. Create a Google Kubernetes Engine cluster and make sure `kubectl` is pointing to the cluster.
    ```bash
    gcloud services enable container.googleapis.com
    gcloud services enable cloudbuild.googleapis.com
    gcloud container clusters create node --enable-autoupgrade --num-nodes 2 --zone us-east4-c
    kubectl get nodes
    ```
3. Enable Google Container Registry (GCR) on your GCP project and configure the `docker` CLI to authenticate to GCR:
   ```bash
   gcloud services enable containerregistry.googleapis.com
   gcloud auth configure-docker -q
	```

## Deployment
### Locally with Docker
Now that we have our tools installed let's go ahead and deploy the application:

```bash
> git clone https://github.com/elatovg/nginx-tomcat-mysql.git
> docker-compose up
...
...
db_1      | 2020-07-30T19:17:01.570130Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
db_1      | 2020-07-30T19:17:01.570533Z 0 [System] [MY-013602] [Server] Channel mysql_main configured to support TLS. Encrypted connections are now supported for this channel.
db_1      | 2020-07-30T19:17:01.575095Z 0 [Warning] [MY-011810] [Server] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is accessible to all OS users. Consider choosing a different directory.
db_1      | 2020-07-30T19:17:01.602519Z 0 [System] [MY-010931] [Server] /usr/sbin/mysqld: ready for connections. Version: '8.0.21'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server - GPL.
```

After it's done it will have all the services up. And if you check for local docker images, you will see a newly built and pulled images:

```bash
> docker images
REPOSITORY                           TAG                 IMAGE ID            CREATED             SIZE
nginx-tomcat-mysql_nginx             latest              d75574be3baf        6 minutes ago       132MB
tomcat                               latest              9a9ad4f631f8        40 hours ago        647MB
mysql                                latest              e3fcc9e1cc04        7 days ago          544MB
nginx                                latest              8cf1bfb43ff5        8 days ago          132MB
```

And you can `curl` the nginx container to see the simple results of what is in the sample table:

```bash
> curl http://localhost/example-webapp/test-datasource.jsp?tab=example_table

<html><body><pre>1 | 2020-07-30 19:16:57.0 | example-1 | value-1
2 | 2020-07-30 19:16:57.0 | example-2 | value-2
3 | 2020-07-30 19:16:57.0 | example-3 | value-3
4 | 2020-07-30 19:16:57.0 | example-4 | value-4
5 | 2020-07-30 19:16:57.0 | example-5 | value-5
6 | 2020-07-30 19:16:57.0 | example-6 | value-6
7 | 2020-07-30 19:16:57.0 | example-7 | value-7
8 | 2020-07-30 19:16:57.0 | example-8 | value-8
9 | 2020-07-30 19:16:57.0 | example-9 | value-9

</pre></body></html>%
```
### Clean Up

To stop the containers, you can back to the original terminal and just type **Ctrl-C** and it will stop all of the containers:

```bash
nginx_1   | 172.18.0.1 - - [30/Jul/2020:19:24:00 +0000] "GET /example-webapp/test-datasource.jsp?tab=example_table HTTP/1.1" 200 472 "-" "curl/7.64.1" "-"
^CGracefully stopping... (press Ctrl+C again to force)
Stopping nginx-tomcat-mysql_nginx_1  ... done
Stopping nginx-tomcat-mysql_tomcat_1 ... done
Stopping nginx-tomcat-mysql_db_1     ... done
```
You will see both of the containers stopped:

```bash
> docker ps -a
CONTAINER ID        IMAGE                      COMMAND                  CREATED             STATUS                        PORTS               NAMES
e2255643c253        nginx-tomcat-mysql_nginx   "/docker-entrypoint.…"   8 minutes ago       Exited (0) 22 seconds ago                         nginx-tomcat-mysql_nginx_1
d2b1d9914e81        tomcat                     "catalina.sh run"        8 minutes ago       Exited (143) 22 seconds ago                       nginx-tomcat-mysql_tomcat_1
3d17c7c63ed8        mysql:latest               "docker-entrypoint.s…"   8 minutes ago       Exited (0) 20 seconds ago                         nginx-tomcat-mysql_db_1
```

If you like, you can remove the containers. When you delete the **mysql** container you also remove the test database.

```bash
> docker rm $(docker ps -a -q)
e2255643c253
d2b1d9914e81
3d17c7c63ed8
```

Or from the directly where your `docker-compose.yml` files resides you can run the following:

```bash
 > docker-compose rm -f
Going to remove nginx-tomcat-mysql_nginx_1, nginx-tomcat-mysql_tomcat_1, nginx-tomcat-mysql_db_1
Removing nginx-tomcat-mysql_nginx_1  ... done
Removing nginx-tomcat-mysql_tomcat_1 ... done
Removing nginx-tomcat-mysql_db_1     ... done
```

You can also delete the created and pulled docker images:

```bash
 > docker rmi nginx-tomcat-mysql_nginx tomcat mysql nginx
Untagged: nginx-tomcat-mysql_nginx:latest
Deleted: sha256:d75574be3baff3527645ddf5de4f24e07e6876e99abe7c3c3bf9a57070f0f736
...
```

### Remotely on GKE (TODO)