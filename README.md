# MongoDB_Replica

Running Mongo DB Replica Set on three different server by Docker Swarm.

The oplog size sets to 3 MB, because of 5% of free disk space.

### Initialize Manager Node

    docker swarm init --listen-addr *manager-ip:2377* --advertise-addr *manager-ip*

*Output*

    $ docker swarm init --advertise-addr *manager-ip*
    Swarm initialized: current node (wvkk65249ch286431df31) is now a manager.

    To add a worker to this swarm, run the following command:

        docker swarm join \
        --token *token* \
        *manager-ip*

    To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.

### Join Worker Nodes into network

    docker swarm join --token *token* *manager-ip*

### Setup Mongo DB on each node

First, you should update label on each node :

    ssh <-- The manager node server -->
    docker node update --label-add replica=1 *manager-ip*
    docker node update --label-add replica=2 *worker1-ip*
    docker node update --label-add replica=3 *worker2-ip*

Second, deploy service on manager node :

    ssh <-- The manager node server -->
    docker stack deploy -c docker-stack.yml overlay

And, you should login *Manager node* and run the under command :

    ./initiate.sh

    # // if publish port was set before, need to remove first and update it.
    # docker service update --publish-rm 27017:27017 mongo_replica_mongo1

    docker service update --publish-add 27017:27017 mongo_replica_mongo1

## Persistent storage

Data is stored at `Docker Swarm Volume`, if you want to check the `Mountpoint` on physical machine. Running the following command:

    docker volume inspect *service name*

Data will be persistent between service runs. To remove docker stack and all data. Run the following command:

    docker stack rm overlay
    docker volume rm $(docker volume ls -qf label=com.docker.stack.namespace=mongo_replica)

Leave smarm mode

    docker swarm leave

## Run example project

Let's run Koalab :

    docker service create --env 'MONGO_URL=mongodb://mongo1:27017,mongo2:27017,mongo3:27017/koalab?replicaSet=rs1' --name koalab --network mongo --replicas 2 --publish 8080:8080 kalahari/koalab

## Reference

- [Docker Swarm Document](https://docs.docker.com/engine/reference/commandline/swarm_init)
- [Learning note about Docker Swarm Mode](http://www.evanlin.com/til-2016-07-13/)
- [Mongo Documentation - mongod](https://docs.mongodb.com/manual/reference/program/mongod/)
- [Running a MongoDB Replica Set on Docker 1.12 Swarm Mode: Step by Step](https://medium.com/@kalahari/running-a-mongodb-replica-set-on-docker-1-12-swarm-mode-step-by-step-a5f3ba07d06e)

---

## Support

[![ko-fi](https://www.ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/D1D1WGU9)

<div><a href="https://payping.ir/@hatamiarash7"><img src="https://cdn.payping.ir/statics/Payping-logo/Trust/blue.svg" height="128" width="128"></a></div>

## Contributing

1. Fork it !
2. Create your feature branch : `git checkout -b my-new-feature`
3. Commit your changes : `git commit -am 'Add some feature'`
4. Push to the branch : `git push origin my-new-feature`
5. Submit a pull request :D

## Issues

Each project may have many problems. Contributing to the better development of this project by reporting them.
