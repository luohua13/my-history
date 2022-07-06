# istioctl ps 原理
istioctl ps --vklog=9999
 ## 寻找 Istiod pod
```bash
curl -v -XGET  -H "Accept: application/json, */*" -H "User-Agent: istioctl/1.12.0" 'https://192.168.131.241:6443/api/v1/namespaces/istio-system/pods?fieldSelector=status.phase%3DRunning&labelSelector=app%3Distiod'
```
```bash
curl -v -XGET  -H "Accept: application/json, */*" -H "User-Agent: istioctl/1.12.0" 'https://192.168.131.241:6443/api/v1/namespaces/istio-system/pods/istiod-1-12-67c5497865-bkvq4'
```
## 执行 portforward
```bash
curl -v -XPOST  -H "X-Stream-Protocol-Version: portforward.k8s.io" -H "User-Agent: istioctl/1.12.0" 'https://192.168.131.241:6443/api/v1/namespaces/istio-system/pods/istiod-1-12-67c5497865-bkvq4/portforward'
```

