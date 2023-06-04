# mongodb-replicaset-docker

Complete guide for MongoDB Replication with docker compose and docker swarm.
- Modify mongo docker image version that suits your environment.
- Change `example.env` filename to `.env`, add your own username and password for root user.

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
1. Prepare three different instances.
2. Add docker secret to manage replica key.
```
$ docker secret create mongo-replica-key ./replica.key
```
3. Enable docker swarm.
   1. Create a swarm.
   `docker swarm init`
   2. Add nodes to the swarm.
   `docker swarm join-token worker` will give you a complete command to join with token. Copy and paste to each instance that will become worker nodes.
   3. Add labels to each node.
   `docker node update --label-add LABEL_INFO NODE_NAME`
   For example, run `docker node update --label-add name=db MANAGER_NODE` command to add `name-db` label.
4. Deploy service.
   `docker stack deploy -c docker-compose.yml STACK_NAME`
   If you set `STACK_NAME` to `mongo`, it will create three services.
   ```
   docker service ls
    ID             NAME           MODE         REPLICAS   IMAGE         PORTS
    wg59os2v9ifb   mongo_mongo1   replicated   1/1        mongo:4.2.3   *:27017->27017/tcp
    ioo44nri3zj2   mongo_mongo2   replicated   1/1        mongo:4.2.3   *:27018->27017/tcp
    v0hvhczngyik   mongo_mongo3   replicated   1/1        mongo:4.2.3   *:27019->27017/tcp
   ```
5. Set Replication setting at Primary Node.
    ```
    docker exec -it mongo_mongo1 mongo -u <USERNAME> -p <PWD>

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