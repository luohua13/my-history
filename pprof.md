
# 调试 kubernetes pod
```bash
kubectl port-forward pod-123ab -n a-namespace 6060
curl "http://127.0.0.1:6060/debug/pprof/profile?seconds=600" > cpu.pprof
```
