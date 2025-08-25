# MySQL 8.4 Group Replication

This is just yet another example of a fully functional MySQL 8.4 Group Replication cluster using Docker Compose. \
It provisions five MySQL server instances in a single-primary topology and includes MySQL Router 8.4 for transparent read-write and read-only routing.

## Usage

Launch the MySQL servers:
```shell
docker-compose up -d
```

### Initializing Group Replication

*On the first run, the cluster has to be bootstrapped manually.*

Connect to any MySQL instance:
```shell
docker compose run mysql1 bash
```

Run MySQL Shell in JS mode:
```shell
mysqlsh --js
```

Run the next commands in sequence:
```javascrpit
// Create the cluster and add the first instance
let cluster = dba.createCluster('group-replication-cluster', { localAddress: "mysql1:33061", communicationStack: "XCOM" });

// Add secondary instances
cluster.addInstance('root:password@mysql2:3306', { recoveryMethod: "clone", localAddress: "mysql2:33061" });
cluster.addInstance('root:password@mysql3:3306', { recoveryMethod: "clone", localAddress: "mysql3:33061" });
cluster.addInstance('root:password@mysql4:3306', { recoveryMethod: "clone", localAddress: "mysql4:33061" });
cluster.addInstance('root:password@mysql5:3306', { recoveryMethod: "clone", localAddress: "mysql5:33061" });
```

## Verifying Cluster Status

After bootstrapping, the status of the cluster can be verified from MySQL Shell with:
```javascript
dba.getCluster('group-replication-cluster').status()
```

One of the instances should be `PRIMARY` and four `SECONDARY`, all reporting `ONLINE`.

## Restarting the Cluster

### Graceful recovery
If the cluster was stopped without deletion of volumes, it can be restarted with the following command from MySQL Shell:

```javascript
dba.rebootClusterFromCompleteOutage();
```

### Automatic bootstrapping

Alternatively, automatic start-on-boot can be enabled.

In `gr-common.cnf` set:
```conf
loose-group_replication_start_on_boot=ON
```

In `gr-node1.cnf` add:
```conf
loose-group_replication_bootstrap_group=ON
```

## Deploying MySQL Router

MySQL Router can be deployed with a separate compose file:
```shell
docker-compose -f router.yaml up -d
```

By default, Router exposes ports:
- `6446` read-write
- `6447` read-only

## Cleanup

```shell
docker-compose down
docker-compose -f router.yaml down  -v
```