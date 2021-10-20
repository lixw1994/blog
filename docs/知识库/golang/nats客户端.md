## 配置项
```go
nc, err := nats.Connect(strings.Join(addrs, ","),
		nats.Name(name),
		nats.Token(token),

		nats.Timeout(time.Second*3),
		nats.PingInterval(5*time.Second),
		nats.MaxPingsOutstanding(5),
		nats.NoEcho(),

		nats.MaxReconnects(100),
		nats.ReconnectWait(time.Second),
		nats.ReconnectHandler(func(nc *nats.Conn) {
			// handle reconnect event
		}),
		// 10MB
		nats.ReconnectBufSize(10*1024*1024),

		nats.FlusherTimeout(time.Second*3),
		nats.ClosedHandler(func(c *nats.Conn) {
		}),
		nats.DisconnectErrHandler(func(nc *nats.Conn, err error) {
			// handle disconnect error event
		}),
		nats.DrainTimeout(time.Second*5),
	)
```

## 优雅推出
- 使用`Drain`而不是Close
- 进程需要保证`DrainTimeout`之后在`Exit`

## 概念上分离发布者和订阅者
- 同一个`Conn`的操作内部会加锁，所以`Conn`控制的范围应该小
- 同一个`Conn`发布订阅统一主题，可能死锁
- 一个项目可能有多个不同的功能模块，不同模块对QPS很有可能大不相同，建议使用多个`Conn`，每个`Conn`的统计就会变得很有意义。

## fan-in -> workers
- 订阅者包含处理能力的属性
- 每个订阅者都应该先收集消息，然后分发给`worker`进行处理，这样代码比较清晰，而且可以精确控制并发量(处理能力)。
- 如果遇到很特殊的主题，说明需要拆出来一个特殊的订阅者。

## 通讯协议
- `protobuf`
- `json`