# mongo-replicaset-docker
Setting MongoDB replica set with docker compose

1. Create your own keyfile. Key file should be under scripts directory. (or you can change volumes field)
```
openssl rand -base64 700 > file.key
chmod 400 file.key
```

2. Copy keyfile into each mongo containers
```
volumes:
    - "./scripts/file.key:/data/fle.key"
```

3. Run each container
```
docker-compose up -d
```

    Run `setup.sh` script in `mongo1` container to set a replica set.
```
$ docker exec -it mongo1 bash
$ cd scripts
$ ./setup.sh
```