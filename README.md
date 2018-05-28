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

### create bridge network
``` bash
docker network create --driver bridge cluster-net
```

### create MON
``` bash
docker run -d --net=cluster-net -v ${VOLUMES_PATH}/ceph:/etc/ceph/ -v ${VOLUMES_PATH}/lib/ceph/:/var/lib/ceph/ -e MON_IP=172.18.0.2 -e CEPH_PUBLIC_NETWORK=172.18.0.0/16 --name mon1 ceph/daemon mon
```

### create MGR
``` bash
docker run -d --net=cluster-net --name=mgr -v ${VOLUMES_PATH}/ceph:/etc/ceph -v ${VOLUMES_PATH}/lib/ceph/:/var/lib/ceph -p 7000:7000 ceph/daemon mgr
```

### create OSD
``` bash
docker run -d --net=cluster-net --privileged=true --pid=host -v ${VOLUMES_PATH}/ceph:/etc/ceph -v ${VOLUMES_PATH}/lib/ceph/:/var/lib/ceph/ -v /dev/:/dev/ -e OSD_DEVICE=/dev/sdb -e OSD_TYPE=disk -e OSD_FORCE_ZAP=1 --name osd1 ceph/daemon osd
```

### create RGW
``` bash
docker run -d --net=cluster-net -v ${VOLUMES_PATH}/lib/ceph/:/var/lib/ceph/ -v ${VOLUMES_PATH}/ceph:/etc/ceph -p 8080:8080 --name rgw1 ceph/daemon rgw
```

### create RGW user
``` bash
docker exec -ti rgw1 radosgw-admin user create --uid="uid" --display-name="testName" --email="test@example.com"
```

### create MDS
``` bash
docker run -d --net=cluster-net -v ${VOLUMES_PATH}/lib/ceph/:/var/lib/ceph/ -v ${VOLUMES_PATH}/ceph:/etc/ceph -e CEPHFS_CREATE=1 --name mds1 ceph/daemon mds
```