## Docker Swarm

集群的管理和编排，类似简单版本的k8s。

**集群搭建**

docker swarm init

docker swarm join

**创建服务**

docker service create

#### raft一致性协议

节点分为manage节点和worker节点。

manage节点至少3台保证高可用。