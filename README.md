# Run in Docker-Compose

### create bridge network
``` bash
docker network create --driver bridge ceph-cluster-net
```

### copy `env-example` to `.env`
```
cp env-example .env
```

### set your environment in `.env`
```
vim .env
```

### up all services
``` bash
docker-compose up -d
```

# Run in Docker Command

### read .env file
``` bash
source .env
```

### create bridge network
``` bash
docker network create --driver bridge ceph-cluster-net
```

### create MON
``` bash
docker run -d --net=ceph-cluster-net --name mon1 -v ${VOLUMES_PATH}/ceph:/etc/ceph/ -v ${VOLUMES_PATH}/lib/ceph/:/var/lib/ceph/ -e MON_IP=${MON1_IP} -e CEPH_PUBLIC_NETWORK=${MON1_CEPH_PUBLIC_NETWORK} ceph/daemon:${CEPH_CONTAINER_VERSION} mon
```

### create MGR
``` bash
docker run -d --net=ceph-cluster-net --name=mgr -v ${VOLUMES_PATH}/ceph:/etc/ceph -v ${VOLUMES_PATH}/lib/ceph/:/var/lib/ceph -p ${DASHBOARD_PORT}:7000 ceph/daemon:${CEPH_CONTAINER_VERSION} mgr
```

### create OSD
``` bash
docker run -d --net=ceph-cluster-net --name osd1 --privileged=true --pid=host -v ${VOLUMES_PATH}/ceph:/etc/ceph -v ${VOLUMES_PATH}/lib/ceph/:/var/lib/ceph/ -v /dev/:/dev/ -e OSD_DEVICE=${OSD1_DEVICE} -e OSD_TYPE=disk -e OSD_FORCE_ZAP=1 ceph/daemon:${CEPH_CONTAINER_VERSION} osd
```

### create RGW
``` bash
docker run -d --net=ceph-cluster-net --name rgw1 -v ${VOLUMES_PATH}/lib/ceph/:/var/lib/ceph/ -v ${VOLUMES_PATH}/ceph:/etc/ceph -p ${RGW_PORT}:8080 ceph/daemon:${CEPH_CONTAINER_VERSION} rgw
```

### create RGW user
``` bash
docker exec -ti rgw1 radosgw-admin user create --uid="uid" --display-name="testName" --email="test@example.com"
```

### create MDS
``` bash
docker run -d --net=ceph-cluster-net --name mds1 -v ${VOLUMES_PATH}/lib/ceph/:/var/lib/ceph/ -v ${VOLUMES_PATH}/ceph:/etc/ceph -e CEPHFS_CREATE=1 ceph/daemon:${CEPH_CONTAINER_VERSION} mds
```