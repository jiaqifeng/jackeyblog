# 架构
大概画个草图:
![arch](/pinpoint-arch.png)

# 通信
agent和collector之间通信的数据格式是thrift编码,具体在源码的thrift目录下.

TCP,默认端口9994,agent发送除stat和span之外的信息,collector会发送一些命令信息(收集动态流量?).

UDP,默认端口9995,发送定时的jvm各种统计信息(stat),默认每5秒收集一次,30秒发送一次,配置中有相关配置项.

UDP,默认端口9996,发送span信息.

# 数据结构
## span的结构
# hbase
collector会将每个收到span信息,才分成多个对hbase的insert/update异步操作.
# 版权
gojs是商业版权,?eChart?也是商业版权
# FAQ
## 
