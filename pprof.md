
Profiling 是指在程序执行过程中，收集能够反映程序执行状态的数据。
火焰图（flame graph）是性能分析的利器,在go1.1之前的版本我们需要借 go-torch 生成, 在go1.1后go tool pprof集成了此功能

# 使用流程
## 1、获取 cpuprofile
获取最近10秒程序运行的cpuprofile,-seconds参数不填默认为30。
```bash
go tool pprof http://127.0.0.1:8080/debug/pprof/profile -seconds 10
```
等10s后会生成一个: pprof.samples.cpu.001.pb.gz文件

## 2、生成火焰图
```bash
go tool pprof -http=:8081 ~/pprof/pprof.samples.cpu.001.pb.gz
```

# 调试 kubernetes pod
```bash
kubectl port-forward pod-123ab -n a-namespace 6060
curl "http://127.0.0.1:6060/debug/pprof/profile?seconds=600" > cpu.pprof
# 本地查看
go tool pprof -http=:8081 cpu.pprof
```
# on-cpu as well as off-cpu(eg. I/O) time
https://github.com/felixge/fgprof
```golang
import(
	_ "net/http/pprof"
	"github.com/felixge/fgprof"
)

func main() {
	go func() {
		http.DefaultServeMux.Handle("/debug/fgprof", fgprof.Handler())
		log.Println(http.ListenAndServe("localhost:6060", nil))
	}()
}
```
