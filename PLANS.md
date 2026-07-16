# XHTTP 移植状态

本文记录当前工作区中 XHTTP 的实现边界。目标是以 sing-box 的结构独立重写
Xray XHTTP 的协议行为；没有复制 Xray-core 的源文件或代码片段，新增代码按
sing-box 的 GPL-3.0 项目规则维护。

## 本次已实现

### 集成与配置

- 注册新的 V2Ray transport：`type: "xhttp"`。
- 新增 `V2RayXHTTPOptions`、随机区间和 XMUX 选项的数据模型，并接入 JSON
  编解码。
- 接入现有 VMess、VLESS、Trojan 的通用 V2Ray transport 工厂；三者无需各自
  修改数据面。
- 增加中英文配置文档和当前能力边界说明。

### 协议核心

- 支持 `stream-one`、`stream-up`、`packet-up`；`auto` 当前选择 `packet-up`。
- 支持 path/query/header/cookie 中的 session ID 与 sequence placement。
- 支持 legacy 与 obfs XPadding，含 `repeat-x` 与 `tokenish`。
- 支持 body/header/cookie/auto 上行载荷，header/cookie 使用 URL-safe Base64。
- 服务端支持有界 packet 队列、按 sequence 重组、30 秒未连接 session 回收、
  单请求大小限制和请求头大小限制。
- 使用分离的读写流把 XHTTP 映射成 `net.Conn`，以复用既有入站协议处理。

### HTTP 与性能基础

- 支持 HTTP/1.1、TLS HTTP/2 和 h2c 的 client/server 代码路径。
- HTTP/1.1 stream-one 使用 Go 的 full-duplex 响应控制，避免服务端先耗尽请求
  body 才开始下行响应。
- packet-up 写入使用有界队列、分片、合并、最小发送间隔和共享 HTTP transport；
  H1 transport 保持 idle connection pool，H2 transport 保持可复用的多路连接。
- TLS client 使用 sing-box 的 TLS/dialer 抽象，而非强制标准库 TLS，保留
  uTLS/REALITY 的 H2 接入路径。

## 已验证

- `transport/v2rayxhttp` 的本地端到端回环测试已覆盖三个模式：
  `stream-one`、`stream-up`、`packet-up`。
- 已通过 `go test -race ./transport/v2rayxhttp`。
- 在 Go 1.24.7 下已通过 `go test ./...` 和 `go build ./cmd/sing-box`。
- 当前环境执行 Go 1.24.7 时使用：

  ```bash
  GOSUMDB=sum.golang.org GOTOOLCHAIN=go1.24.7 <go command>
  ```

## 尚未实现或尚未完成验证

### 协议与功能

- HTTP/3 / QUIC（`with_quic`）尚未实现。
- `download_settings` 尚未设计为 sing-box 原生选项，也尚未实现独立下行目标。
- `auto` 尚未按 Xray 的 REALITY/`downloadSettings` 规则自动选择
  `stream-one` 或 `stream-up`。
- XMUX 选项已进入配置模型，但尚未实现 Xray 等价的多 HTTP client 池、请求预算、
  reuse 次数、并发上限和可复用时长调度。
- `uplink_chunk_size` 尚未用于 header/cookie 的动态分块；当前采用固定 3000 字节
  的编码分块。
- `sc_stream_up_server_secs` 尚未实现 Xray 的上行响应 padding 保活行为。
- 服务端响应 padding 尚未实现；目前只校验客户端请求 padding。
- Unix domain socket、浏览器 dialer、UDP hopping、QUIC 拥塞控制等 Xray 特性
  尚未纳入范围。

### 互通、性能与稳定性

- 尚未完成 Xray client → sing-box server 和 sing-box client → Xray server 的
  双向互通矩阵。
- 尚未完成 VMess、VLESS、Trojan 的完整外层集成测试矩阵。
- 尚未建立与 Xray 同机对照的吞吐、P99 延迟、连接数、CPU、`allocs/op` 基准。
- 尚未执行长期连接、乱序/丢包边界、session 回收、goroutine/连接泄漏的压力测试。
- HTTP/2、REALITY/uTLS 的实际端到端组合仍需专门验证。

## 下一步优先级

1. 建立 Xray 双向互通测试，先覆盖默认 padding、三种 mode、body/header/cookie
   上行和 HTTP/2。
2. 完成 XMUX manager 与 request/reuse 预算，随后做基准和 pprof 对照。
3. 补齐 HTTP/2 + TLS、REALITY/uTLS 的端到端测试。
4. 设计并实现 `download_settings`。
5. 在 `with_quic` 下实现 HTTP/3，并补充 QUIC 参数与性能验证。
