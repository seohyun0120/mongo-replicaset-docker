# mongodb-replicaset-docker

Complete guide for MongoDB Replication with docker compose and docker swarm. 

## Docker Compose Mode
1. Create your own keyfile. Key file should be under scripts directory. (or you can change volumes field)
```
openssl rand -base64 700 > file.key
chmod 400 file.key
```

2. Copy keyfile into each mongo containers using volumes option.
```
volumes:
    - ./scripts/file.key:/opt/key/replica.key
```

3. Run 3 services using docker compose.
```
docker-compose -f docker-compse-local.yml up -d
```

4. Enter `mongo1` container, run mongo shell command, add replica setting.
You can just copy `scripts/replica-set.sh`.
```
docker exec -it mongo1 mongo -u <USERNAME> -p <PWD>

rs.status() // check replica setting

rs.initiate({
    "_id": "rs0",
    "members": [
        {
            "_id": 0,
            "host": "mongo1:27017",
            "priority": 2
        },
        {
            "_id": 1,
            "host": "mongo2:27017",
            "priority": 0.5
        },
        {
            "_id": 2,
            "host": "mongo3:27017",
            "priority": 0.5
        }
    ]
});
```

## Docker Swarm Mode
