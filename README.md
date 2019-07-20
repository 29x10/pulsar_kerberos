# Pulsar Kerberos Setup

## Before you setup

Assume I deploy all of them within same host

* hostname as jeromex.pulsar.com
* user id as jeromex
* keytab location /tmp/jeromex.keytab
* keytab principle jeromex/jeromex.pulsar.com@realm
* ticket cache principle jeromex@realm
* pulsar binary home /tmp/pulsar

## Zookeeper

### Change details please refer to commit

```
export OPTS="-Djava.security.auth.login.config=/tmp/pulsar/jaas/zookeeper_server_jaas.conf"
/tmp/pulsar/bin/pulsar zookeeper
```
### Verify zookeeper SASL authentication

```
export OPTS="-Djava.security.auth.login.config=/tmp/pulsar/jaas/zookeeper_client_jaas.conf -Dzookeeper.sasl.client.username=jeromex"
/tmp/pulsar/bin/pulsar zookeeper-shell -server jeromex.pulsar.com:2181
```

## Bookkeeper

### Change details please refer to commit

```
export OPTS="-Djava.security.auth.login.config=/tmp/pulsar/jaas/bookie_jaas.conf -Dzookeeper.sasl.client.username=jeromex -Dbookkeeper.sasl.servicename=jeromex"
/tmp/pulsar/bin/bookkeeper shell metaformmat
/tmp/pulsar/bin/pulsar bookie
```

## Broker

### Change details please refer to commit
```
export OPTS="-Djava.security.auth.login.config=/tmp/pulsar/jaas/broker_jaas.conf -Dzookeeper.sasl.client.username=jeromex -Dbookkeeper.sasl.servicename=jeromex"
/tmp/pulsar/bin/pulsar broker
```

### Create cluster, tenant, namespace

```
export OPTS="-Djava.security.auth.login.config=/tmp/pulsar/jaas/broker_jaas.conf -Dzookeeper.sasl.client.username=jeromex -Dbookkeeper.sasl.servicename=jeromex"
/tmp/pulsar/bin/pulsar-admin clusters create jeromex_cluster --url http://jeromex.pulsar.com:8080  --broker-url pulsar://jeromex.pulsar.com:6650
/tmp/pulsar/bin/pulsar-admin tenants create jeromex-tenant -r jeromex/jeromex.pulsar.com@realm,jeromex@realm -c jeromex_cluster
/tmp/pulsar/bin/pulsar-admin namespaces create jeromex-tenant/jeromex-namespace --clusters jeromex_cluster
export OPTS="-Djava.security.auth.login.config=/tmp/pulsar/jaas/client_jaas.conf"
/tmp/pulsar/bin/pulsar-client produce persistent://jeromex-tenant/jeromex-namespace --messages "hello pulsar"
```