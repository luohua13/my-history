
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
# On-cpu as well as off-cpu(eg. I/O) time
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
# 工具型应用开启 pprof
## CPU
```golang
import "runtime/pprof"
func main() {
    file, _ := os.Create("./cpu.pprof") // 在当前路径下创建一个cpu.pprof文件
    pprof.StartCPUProfile(file) // 往文件中记录CPU profile信息
    defer func() {
        // 退出之前 停止采集
        pprof.StopCPUProfile()
        file.Close()
    }()
}
```

## Heap
```golang
file, _ := os.Create("./mem.pprof")
pprof.WriteHeapProfile(file)
f2.Close()
```
# Other
- 当 CPU 性能分析启用后，Go runtime 会每 10ms 就暂停一下，记录当前运行的 goroutine 的调用堆栈及相关数据。当性能分析数据保存到硬盘后，我们就可以分析代码中的热点了。
- 内存性能分析则是在堆（Heap）分配的时候，记录一下调用堆栈。默认情况下，是每 1000 次分配，取样一次，这个数值可以改变。栈(Stack)分配 由于会随时释放，因此不会被内存分析所记录。由于内存分析是取样方式，并且也因为其记录的是分配内存，而不是使用内存。因此使用内存性能分析工具来准确判断程序具体的内存使用是比较困难的。
- 阻塞分析是一个很独特的分析，它有点儿类似于 CPU 性能分析，但是它所记录的是 goroutine 等待资源所花的时间。阻塞分析对分析程序并发瓶颈非常有帮助，阻塞性能分析可以显示出什么时候出现了大批的 goroutine 被阻塞了。阻塞性能分析是特殊的分析工具，在排除 CPU 和内存瓶颈前，不应该用它来分析。

# 参考
https://juejin.cn/post/6844903992720359432
https://www.lixueduan.com/post/go/pprof/
https://pkg.go.dev/net/http/pprof
https://zhuanlan.zhihu.com/p/71529062
https://danlimerick.wordpress.com/2017/01/24/profiling-golang-programs-on-kubernetes/
