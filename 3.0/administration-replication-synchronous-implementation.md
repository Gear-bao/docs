---
layout: default
description: Synchronous replication can be configured per collection via the property replicationFactor
---
Implementation
==============

### Architecture inside the cluster

Synchronous replication can be configured per collection via the property *replicationFactor*. Synchronous replication requires a cluster to operate.

Whenever you specify a *replicationFactor* greater than 1 when creating a collection, synchronous replication will be activated for this collection. The cluster will determine suitable *leaders* and *followers* for every requested shard (*numberOfShards*) within the cluster. When requesting data of a shard only the current leader will be asked whereas followers will only keep their copy in sync. This is due to the current implementation of transactions.

Using *synchronous replication* alone will guarantee consistency and high availabilty at the cost of reduced performance: Write requests will have a higher latency (due to every write-request having to be executed on the followers) and read requests won't scale out as only the leader is being asked.

In a cluster synchronous replication will be managed by the *coordinators* for the client. The data will always be stored on *primaries*.

The following example will give you an idea of how synchronous operation has been implemented in ArangoDB.

1. Connect to a coordinator via arangosh
2. Create a collection

    127.0.0.1:8530@_system> db._create("test", {"replicationFactor": 2})

3. the coordinator will figure out a *leader* and 1 *follower* and create 1 *shard* (as this is the default)
3. Insert data

    127.0.0.1:8530@_system> db.test.insert({"replication": "😎"})

4. The coordinator will write the data to the leader, which in turn will
replicate it to the follower.
5. Only when both were successful the result is reported to be successful

    { 
        "_id" : "test/7987", 
        "_key" : "7987", 
        "_rev" : "7987" 
    }

   When a follower fails, the leader will give up on it after 3 seconds
   and proceed with the operation. As soon as the follower (or the network
   connection to the leader) is back up, the two will resynchronize and
   synchronous replication is resumed. This happens all transparently
   to the client.

The current implementation of ArangoDB does not allow changing the replicationFactor later. This is subject to change. In the meantime the only way is to create a new collection having the desired `replicationFactor`, copy all data and swap names with the original one.

### Automatic failover

Whenever the leader of a shard is failing and there is a query trying to access data of that shard the coordinator will continue trying to contact the leader until it timeouts. The internal cluster supervision will check cluster health every few seconds and will take action if there is no heartbeat from a server for 15 seconds. If the leader doesn't come back in time the supervision will reorganize the cluster by promoting for each shard a follower that is in sync with its leader to be the new leader. From then on the coordinators will contact the new leader.

The process is best outlined using an example:

1. The leader of a shard (lets name it DBServer001) is going down.
2. A coordinator is asked to return a document of a shard DBServer001 is managing:

    127.0.0.1:8530@_system> db.test.document("100069")

3. The coordinator tries to contact the leader (DBServer001) and timeouts.
4. The coordinator retries to contact the leader (DBServer001) and timeouts.
5. The supervision detects outage of DBServer001.
6. The supervision promotes one of the followers (say DBServer002) that is in sync to be leader and makes DBServer001 a follower.
7. The coordinator retries to contact the leader (DBServer002) and returns the result:

    { 
        "_key" : "100069", 
        "_id" : "test/100069", 
        "_rev" : "513", 
        "replication" : "😎"
    }
8. After a while the supervision declares DBServer001 to be completely dead.
9. A new follower is determined from the pool of DBservers.
10. The new follower syncs its data from the leader and order is restored.

Please note that there may still be timeouts. Depending on when exactly the request has been done (in regard to the supervision) and depending on the time needed to reconfigure the cluster the coordinator might fail with a timeout error!
