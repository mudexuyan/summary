#
1. Quartz是一个任务调度框架

## 基本原理
1. scheduler
2. trigger
3. job
4. jobdetail
## 流程
1. 任务生成线程池，用来管理job
2. 调度器轮询trigger，执行任务
3. 轮询所有misfire的trigger
## 集群
1. 共享数据库
2. 排它锁避免重复执行和资源竞争