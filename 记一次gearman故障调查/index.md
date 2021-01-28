# 记一次gearman故障调查


# 场景

个业务系统无法生成订单,订单报错和 gearman 的 worker 有关，使用 curl 调用接口发现报 500 错误。

# 过程

- curl 调用接口不通
- 登录 gearman 服务器调用接口不通
- 重启 gearman worker 调用不通
- 查看接口服务是由 lighttp 提供
- 直接调用 80 端口不通
- 重启 lighttp 调用成功

# 原因

生物信息升级了 python

