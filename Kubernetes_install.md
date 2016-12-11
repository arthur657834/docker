50.141 k8s-master
50.142 k8s-minion
50.144 etcd

etcd Cluster：
yum install etcd -y

grep -v ^# /etc/etcd/etcd.conf
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"
ETCD_ADVERTISE_CLIENT_URLS="http://10.1.50.144:2379"

systemctl enable etcd
systemctl start etcd

etcdctl member list
etcdctl mk /coreos.com/network/config '{"Network":"172.17.0.0/16"}'
etcdctl get /coreos.com/network/config

K8s Master:
yum -y install kubernetes

egrep -v "^#" /etc/kubernetes/apiserver | grep -v "^$"
KUBE_API_ADDRESS="--insecure-bind-address=0.0.0.0"
KUBE_ETCD_SERVERS="--etcd-servers=http://10.1.50.144:2379"
KUBE_SERVICE_ADDRESSES="--service-cluster-ip-range=10.254.0.0/16"
KUBE_ADMISSION_CONTROL="--admission-control=NamespaceLifecycle,NamespaceExists,LimitRanger,SecurityContextDeny,ServiceAccount,ResourceQuota"
KUBE_API_ARGS="

egrep -v "^#" /etc/kubernetes/controller-manager |grep -v "^$"
KUBE_CONTROLLER_MANAGER_ARGS="--node-monitor-grace-period=10s --pod-eviction-timeout=10s"

egrep -v "^#" /etc/kubernetes/config | egrep -v "^$"
KUBE_LOGTOSTDERR="--logtostderr=true"
KUBE_LOG_LEVEL="--v=0"
KUBE_ALLOW_PRIV="--allow-privileged=false"
KUBE_MASTER="--master=http://10.1.50.141:8080"

systemctl enable kube-apiserver kube-scheduler kube-controller-manager
systemctl start kube-apiserver kube-scheduler kube-controller-manager

curl http://10.1.50.141:8080/
http://10.1.50.141:8080/swagger-ui/
kubectl get pods


K8s Minions:
yum -y install kubernetes docker flannel bridge-utils

egrep -v "^#" /etc/kubernetes/config | grep -v "^$"
KUBE_LOGTOSTDERR="--logtostderr=true"
KUBE_LOG_LEVEL="--v=0"
KUBE_ALLOW_PRIV="--allow-privileged=false"
KUBE_MASTER="--master=http://10.1.50.141:8080"

egrep -v "^#" /etc/kubernetes/kubelet | grep -v "^$"
KUBELET_ADDRESS="--address=127.0.0.1"
KUBELET_HOSTNAME="--hostname-override=10.1.50.142"
KUBELET_API_SERVER="--api-servers=http://10.1.50.141:8080"
KUBELET_POD_INFRA_CONTAINER="--pod-infra-container-image=registry.access.redhat.com/rhel7/pod-infrastructure:latest"
KUBELET_ARGS=""

systemctl enable kubelet kube-proxy docker
systemctl start kubelet kube-proxy docker

kubectl get nodes


