# Introduction

This docker-compose for deploy ceph version 13.2.1 & Container version v3.1.0-stable-3.1-mimic-centos-7

## Setup

[more info](http://docs.ceph.com/docs/mimic/mgr/dashboard)

## Quick Start

1. configuration `.env`
``` bash
cp env-example .env && vim .env
```

2. create bridge network
``` bash
docker network create --driver bridge ceph-cluster-net
```

3. up monitor and manager
``` bash
docker-compose up -d mon1 mgr
```

4. chnage max object namespace len 
``` bash
vim ${VOLUMES_PATH}/ceph/ceph.conf
```
``` conf
osd pool default min size = 2
max open files = 655350
cephx cluster require signatures = false
cephx service require signatures = false

osd max object name len = 256
osd max object namespace len = 64
```

5. restart monitor and manager
``` bash
docker-compose restart mon1 mgr
```

6. up all Object Storage Daemon
``` bash
docker-compose up -d osd1 osd2 osd3
```

7. up RADOS Gateway
``` bash
docker-compose up -d rgw1
```

8. up Metadata Server
``` bash
docker-compose up -d mds1
```

9. enable `mgr` dashboard module
``` bash
docker-compose exec mon1 ceph mgr module enable dashboard
```

10. create dashboard signed cert 
``` bash
docker-compose exec mon1 ceph dashboard create-self-signed-cert
```

11.  restart `mgr` dashboard module
``` bash
docker-compose exec mon1 ceph mgr module disable dashboard
``` 
``` bash
docker-compose exec mon1 ceph mgr module enable dashboard
```

12. bind dashport port and domain
``` bash
docker-compose exec mon1 ceph config set mgr mgr/dashboard/server_addr mgr
```
``` bash
docker-compose exec mon1 ceph config set mgr mgr/dashboard/server_port 8443
```

13.  get all mgr register services
``` bash
docker-compose exec mon1 ceph mgr services
```

14. create dashboard account
``` bash
docker-compose exec mon1 ceph dashboard set-login-credentials <ACCOUNT> <PASSWORD>
```

15. create rados gateway user
``` bash
docker-compose exec mon1 radosgw-admin user create --uid=<UID> --display-name=<DISPLAYNAME> --system
```

16.  bind rados gatway user to dashboard
``` bash
docker-compose exec mon1 ceph dashboard set-rgw-api-access-key <ACCESS_KEY>
```
``` bash
docker-compose exec mon1 ceph dashboard  set-rgw-api-secret-key <SECRET_KEY>
```

## Install

### create bridge network

``` bash
docker network create --driver bridge ceph-cluster-net
```

### edit environment

``` bash
cp env-example .env
```

### set your environment in `.env`

``` bash
vim .env
```

### read `.env` file if using docker cmd

``` bash
source .env
```

### up monitor and manager

`docker-compose`

``` bash
docker-compose up -d mon1 mgr
```

`docker`

``` bash
docker run -d --net=ceph-cluster-net --name=mon1 -v ${VOLUMES_PATH}/ceph:/etc/ceph/ -v ${VOLUMES_PATH}/lib/ceph/:/var/lib/ceph/ -e MON_IP=${MON1_IP} -e CEPH_PUBLIC_NETWORK=${MON1_CEPH_PUBLIC_NETWORK} ceph/daemon:${CEPH_CONTAINER_VERSION} mon
```

``` bash
docker run -d --net=ceph-cluster-net --name=mgr -v ${VOLUMES_PATH}/ceph:/etc/ceph -v ${VOLUMES_PATH}/lib/ceph/:/var/lib/ceph -p ${DASHBOARD_PORT}:${INTERNAL_DASHBOARD_PORT} ceph/daemon:${CEPH_CONTAINER_VERSION} mgr
```

### up all osds

`docker-compose`

``` bash
docker-compose up -d osd1 osd2 osd3
```

`docker`

``` bash
docker run -d --net=ceph-cluster-net --name=osd1 --privileged=true --pid=host -v ${VOLUMES_PATH}/ceph:/etc/ceph -v ${VOLUMES_PATH}/lib/ceph/:/var/lib/ceph/ -v ${OSD_PATH}/osd1:/var/lib/ceph/osd ceph/daemon:${CEPH_CONTAINER_VERSION} osd_directory
```

``` bash
docker run -d --net=ceph-cluster-net --name=osd2 --privileged=true --pid=host -v ${VOLUMES_PATH}/ceph:/etc/ceph -v ${VOLUMES_PATH}/lib/ceph/:/var/lib/ceph/ -v ${OSD_PATH}/osd2:/var/lib/ceph/osd ceph/daemon:${CEPH_CONTAINER_VERSION} osd_directory
```

``` bash
docker run -d --net=ceph-cluster-net --name=osd3 --privileged=true --pid=host -v ${VOLUMES_PATH}/ceph:/etc/ceph -v ${VOLUMES_PATH}/lib/ceph/:/var/lib/ceph/ -v ${OSD_PATH}/osd3:/var/lib/ceph/osd ceph/daemon:${CEPH_CONTAINER_VERSION} osd_directory
```

### up radosgw

`docker-compose`

``` bash
docker-compose up -d rgw1
```

`docker`

``` bash
docker run -d --net=ceph-cluster-net --name=rgw1 -v ${VOLUMES_PATH}/lib/ceph/:/var/lib/ceph/ -v ${VOLUMES_PATH}/ceph:/etc/ceph -p ${RGW_PORT}:8080 ceph/daemon:${CEPH_CONTAINER_VERSION} rgw
```

### up Metadata Server

`docker-compose`

``` bash
docker-compose up -d mds1
```

`docker`

``` bash
docker run -d --net=ceph-cluster-net --name=mds1 -v ${VOLUMES_PATH}/lib/ceph/:/var/lib/ceph/ -v ${VOLUMES_PATH}/ceph:/etc/ceph -e CEPHFS_CREATE=1 ceph/daemon:${CEPH_CONTAINER_VERSION} mds
```

### Create Dashboard account

`docker-compose`

``` bash
docker-compose exec mon1 ceph dashboard set-login-credentials <user_name> <password>
```

`docker`

``` bash
docker exec -ti mon1 ceph dashboard set-login-credentials <user_name> <password>
```

### Create RGW user

`docker-compose`

``` bash
docker-compose exec mon1 radosgw-admin user create --uid="<user_id>" --display-name="<display_name>" --email="<email>"
```

`docker`

``` bash
docker exec -ti mon1 radosgw-admin user create --uid="<user_id>" --display-name="<display_name>" --email="<email>"
```

### show user info and set radosgw api keys

`docker-compose`

``` bash
docker-compose exec mon1 radosgw-admin user info --uid=<user_id>
```

``` bash
docker-compose exec mon1 ceph dashboard set-rgw-api-access-key <access_key>
```

``` bash
docker-compose exec mon1 ceph dashboard set-rgw-api-secret-key <secret_key>
```

`docker`

``` bash
docker exec -ti mon1 radosgw-admin user info --uid=<user_id>
```

``` bash
docker exec -ti mon1 ceph dashboard set-rgw-api-access-key <access_key>
```

``` bash
docker exec -ti mon1 ceph dashboard set-rgw-api-secret-key <secret_key>
```