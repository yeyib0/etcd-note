# etcd-note
etcd学习笔记

## raft模块
- raft节点状态
- 请求投票
- 条目追加
- 日志压缩
- 日志复制
- 集群配置变更

#### raft安全性，可靠性
- 选举安全
- leader仅追加
- 日志匹配
- leader完整性
- 状态机安全

## 存储
- MVCC
- WAL
- B+树
- 租约

## 客户端交互
- KVServer
- WatchServer
- LeaseServer
- ClusterServer
- AuthServer
- MaintenanceServer
- HealthServer
