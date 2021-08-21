## Kerberos

```sh
yum install -y krb5-libs krb5-server krb5-workstation

```

### 使用

```sh
kadmin.local
addprinc -randkey test
ktadd -norandkey -kt test.keytab test
exit

kinit -kt test.keytab test
klist
```



## zookeeper

```sh
export ZOO_DATA_DIR=/var/lib/zookeeper
export ZOO_LOG_DIR=/var/log/zookeeper
export ZK_NODE_LIST=node01,node02,node03
/opt/zfbdp/zookeeper/setup.sh
```

## kafka

```sh
export KAFKA_DATA_DIR=/data03/kafka
export KAFKA_LOG_DIR=/var/log/kafka
export ZK_NODE_LIST=node01,node02,node03
export KAFKA_NODE_LIST=node01,node02,node03
/opt/zfbdp/kafka/setup.sh
```

使用

```sh
./bin/kafka-topics.sh --create --topic topic01 --replication-factor 1 --partitions 1 --zookeeper node02:2181
./bin/kafka-topics.sh --list --zookeeper node03:2181

export KAFKA_OPTS="-Djava.security.auth.login.config=/opt/zfbdp/kafka/config/jaas.conf"

./bin/kafka-console-producer.sh --topic mytopic --broker-list node02:9092
```

### 认证

```sh
./bin/kafka-acls.sh --authorizer-properties zookeeper.connect=node01:2181,node02:2181,node03:2181 --add --allow-principal User:test --producer --topic *

./bin/kafka-acls.sh --authorizer kafka.security.auth.SimpleAclAuthorizer --authorizer-properties zookeeper.connect=node01:2181,node02:2181,node03:2181 --resource-pattern-type prefixed --add --allow-principal User:test --group * --operation all --topic *

./bin/kafka-acls.sh --authorizer-properties zookeeper.connect=node01:2181,node02:2181,node03:2181 --resource-pattern-type prefixed --add --allow-principal User:test --transactional-id * --operation all --topic *

```

