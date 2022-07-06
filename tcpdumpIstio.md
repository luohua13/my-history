## 流程

### 1、获取 containerID
```bash
kubectl describe pod pod-123
```

### 2、pod 所在主机获取容器 pid
```bash
docker inspect {containerID} |grep -i pid
```
### 3、进入到网络空间

```bash
nsenter -n --target {pid}
```

### 4、抓包
traceroute: 注入sidecar 的客户端到 --> 注入 sidecar 的服务端
```bash
tcpdump -i any -w yulin5.cap #直接抓，抓的是客户端sidecar到服务端sidecar的流量
```

```bash
iptables -I OUTPUT -p tcp --dport 80 -j NFLOG # 服务端 sidecar 出来的流量, 
tcpdump -i nflog -w yulin11.cap
```

