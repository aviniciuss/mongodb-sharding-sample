# Sharding with mongodb

> Let's do a minimal setup sharding.

Folder structure
```
- sharding
    - instances
        - configdb
        - shard01
        - shard02
        - shard03
    - logs
    - mongos.conf
    - config_db.conf
```

## Creating config server
```
$ cd sharding
$ cat config_db.conf
  fork=true
  dbpath=./instances/configdb
  logpath=./logs/configdb.log
  logappend=true
  port=27020
$ mongod --configsvr --config config_db.conf
```
![01.png](https://s3-us-west-2.amazonaws.com/sample-sharding-mongodb/01.png)

## Creating routing instance
```
$ cat mongos.conf
  fork=true
  port=27017
  configdb=mycluster:27020
  logpath=./logs/mongos.log
$ mongos --config mongos.conf
```
![02.png](https://s3-us-west-2.amazonaws.com/sample-sharding-mongodb/02.png)

## Creating sharding instances
```
$ mongod --fork --port 30001 --dbpath ./instances/shard01 --logpath ./logs/shard_01.log
```
![03.png](https://s3-us-west-2.amazonaws.com/sample-sharding-mongodb/03.png)

```
$ mongod --fork --port 30002 --dbpath ./instances/shard02 --logpath ./logs/shard_02.log
```
![04.png](https://s3-us-west-2.amazonaws.com/sample-sharding-mongodb/04.png)

```
$ mongod --fork --port 30003 --dbpath ./instances/shard03 --logpath ./logs/shard_03.log
```

## Adding shards in the cluster
```
$ mongo --port 27017
mongos> sh.addShard( "mycluster:30001")
{ "shardAdded" : "shard0000", "ok" : 1 }
mongos> sh.addShard( "mycluster:30002")
{ "shardAdded" : "shard0001", "ok" : 1 }
mongos> sh.addShard( "mycluster:30003")
{ "shardAdded" : "shard0002", "ok" : 1 }
```

## Usage sharding
You should already have a large collection
```
$ mongo --port 27017
mongos> use test
switched to db test
mongos> sh.enableSharding("test");
{ "ok" : 1 }
```

Creating the index to divide the collection
```
mongos> db.hotels.ensureIndex({ "stars": "hashed"});
{
    "raw" : {
    "mycluster:30001" : {
    "createdCollectionAutomatically" : false,
    "numIndexesBefore" : 1,
    "numIndexesAfter" : 2,
    "ok" : 1
}
},
    "ok" : 1
}
mongos> sh.shardCollection("test.hotels",{"stars": "hashed"});
```

## Sharding distributions
```
$ mongo --port 27017
mongos> db.getCollection('hotels').getShardDistribution();
```
![05.png](https://s3-us-west-2.amazonaws.com/sample-sharding-mongodb/05.png)
