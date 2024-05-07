## 初始化EtcdServer
从v3rpc.Server(s *etcdserver.EtcdServer, tls *tls.Config, interceptor grpc.UnaryServerInterceptor, gopts ...grpc.ServerOption)开始阅读，实际上启动方法在embed.StartEtcd(inCfg *Config)
v3rpc的Server方法，初始化grpc的Server
- 配置序列化，可以看到序列化添加了prometheus的**字节总数统计**
- 配置grpc的tls的证书，使用grpc库生成。其中包含未设置的authToken
- 配置unary调用的拦截器
  - rpc调用日志，记录调用状态 newLogUnaryInterceptor
  - 封装一个拦截方法。newUnaryInterceptor
  - prometheus默认的unary调用拦截器
  - 外部传入的拦截器
- 配置stream调用拦截器
  - 监控了leader状态
  - prometheus默认的stream调用拦截器
- 接收消息最大值
- 发送消息最大值
- 最大并发流
  
## KVServer注册
quotaKVServer结构体实现了grpc的KVServer
```golang
type quotaKVServer struct {
	pb.KVServer
	qa quotaAlarmer
}
// 实现service（grpc） KV
// KV的四个方法Range Put DeleteRange Txn
// quotaKVServer结构体实际上调用的KVServer的方法，KVServer调用etcdserver.RaftKV的方法
// EtcdServer实现了etcdserver.RaftKV。这里applyV3 applierV3这个的实现applierV3backend，mvcc.WatchableKV，backend.Backend来读
type kvServer struct {
  // header是etcd中一个通用的rpc参数，LeaseServer，maintenanceServer都有携带
	hdr header
  // KV操作的实现，
	kv  etcdserver.RaftKV
	// maxTxnOps is the max operations per txn.
	// e.g suppose maxTxnOps = 128.
	// Txn.Success can have at most 128 operations,
	// and Txn.Failure can have at most 128 operations.
	maxTxnOps uint
}
// header结构体
type header struct {
	clusterID int64
	memberID  int64
	sg        etcdserver.RaftStatusGetter
	rev       func() int64
}

func newHeader(s *etcdserver.EtcdServer) header {
	return header{
		clusterID: int64(s.Cluster().ID()),
		memberID:  int64(s.ID()),
		sg:        s,
		rev:       func() int64 { return s.KV().Rev() },
	}
}
```
