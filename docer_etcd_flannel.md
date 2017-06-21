yum install -y etcd flannel

yum install -y docker-ce 用docker-ce这个版本不然会有问题

先从etcd开始，简单说就是"distributed key value store"。

etcd集群的3种方式:
* static
* etcd discovery
* DNS discovery

Static:
```shell
vi /etc/etcd/etcd.conf
ETCD_NAME=186
ETCD_DATA_DIR="/var/lib/etcd/186.etcd"
ETCD_LISTEN_PEER_URLS="http://10.1.50.186:2380"
ETCD_LISTEN_CLIENT_URLS="http://10.1.50.186:2379,http://127.0.0.1:2379"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://10.1.50.186:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://10.1.50.186:2379"
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_INITIAL_CLUSTER_TOKEN="etct-fantasy"
ETCD_INITIAL_CLUSTER="174=http://10.1.50.174:2380,186=http://10.1.50.186:2380"

systemcrl start etcd
etcdctl cluster-health
```
etcd discovery:
```shell
curl https://discovery.etcd.io/new?size=3 ==>  3355288768a611f0ace43cb9a6b7952f
etcdctl set /discovery/3355288768a611f0ace43cb9a6b7952f/_config/size 3

vi /etc/etcd/etcd.conf
ETCD_NAME=186
ETCD_DATA_DIR="/var/lib/etcd/186.etcd"
ETCD_LISTEN_PEER_URLS="http://10.1.50.186:2380"
ETCD_LISTEN_CLIENT_URLS="http://10.1.50.186:2379,http://127.0.0.1:2379"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://10.1.50.186:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://10.1.50.186:2379"
ETCD_DISCOVERY=http://10.1.50.174:2379/v2/keys/discovery/3355288768a611f0ace43cb9a6b7952f

grep FLANNEL_ETCD_PREFIX /etc/sysconfig/flanneld ==> /atomic.io/network
etcdctl set /atomic.io/network/config '{ "Network": "10.2.0.0/16" }'
systemctl start flanneld
etcdctl ls /atomic.io/network/subnets
```
flannel启动成功后会生成/run/flannel/docker
```shell
source /run/flannel/docker
docker daemon ${DOCKER_NETWORK_OPTIONS} >> /dev/null 2>&1 &
```
验证互通
```shell
docker run -it busybox
```
