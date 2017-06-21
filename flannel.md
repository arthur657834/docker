Flannel实质上是一种“覆盖网络(overlay network)”，也就是将TCP数据包装在另一种网络包里面进行路由转发和通信，目前已经支持UDP、VxLAN、AWS VPC和GCE路由等数据转发方式。

flannel服务需要先于docker启动。flannel服务启动时主要做了以下几步的工作：
* 从etcd中获取network的配置信息
* 划分subnet，并在etcd中进行注册
* 将子网信息记录到/run/flannel/subnet.env中
* 之后将会有一个脚本将subnet.env转写成一个docker的环境变量文件/run/flannel/docker


Flannel有两种不同的后端，前面配置的是UDP后端，那是比较慢的方案，因为所有的包都是在用户空间中封装的。VxLAN后端使用Linux内核VxLAN支持，一些硬件特性来实现更快的网络 。
非常容易切换到VxLAN后端，在配置Etcd时，在定义 backend时使用 vxlan:

```shell
./etcdctl set /coreos.com/network/config '{ "Network": "10.0.0.0/8","SubnetLen": 20, "SubnetMin": "10.10.0.0","SubnetMax": "10.99.0.0","Backend": { "Type": "udp", "Port": 7890 } }'

./etcdctl set /coreos.com/network/config '{ "Network": "10.0.0.0/8","SubnetLen": 20, "SubnetMin": "10.10.0.0","SubnetMax": "10.99.0.0","Backend": { "Type": "vxlan", "Port": 7890 } }'
```
