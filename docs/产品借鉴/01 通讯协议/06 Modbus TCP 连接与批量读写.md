# Modbus TCP 连接与批量读写借鉴

核查日期：2026-10-24。本文为 GitHub 与官方资料研究，不是代码实现或硬件实测。

.NET 是用户已确认的实现方向；下述具体库和设计仍是候选建议。Windows x64、Linux x64、Linux ARM64、Windows ARM64 必须逐项验证，不能由“.NET Standard”推导出驱动和全部功能已经兼容。

## 结论

沿用 RTU 的标签模型与应用层映射，替换传输和事务匹配部分。**Modbus TCP 与透明传输 RTU-over-TCP 不相同**，前者有 MBAP 头，后者可能仍承载 RTU 帧及 CRC，配置入口不能混为一种。

## GitHub 候选比较

| 项目 | 已核实能力和许可 | 适用判断 |
| --- | --- | --- |
| [NModbus](https://github.com/NModbus/NModbus) | MIT；RTU、ASCII、TCP、UDP；串口适配有独立项目 | 优先候选，接口层清晰，可将传输与协议隔开；不要选到已沉寂的 NModbus4 原分支 |
| [FluentModbus](https://github.com/Apollo3zehn/FluentModbus) | MIT；TCP/RTU 客户端和服务端，常用功能码与部分扩展 | 轻量对照候选，特别适合搭建受控模拟端；必须核对目标设备所需功能码 |
| [TouchSocket.Modbus](https://github.com/RRQM/TouchSocket) | 当前公开 README 列 RTU/ASCII/TCP 主站模块 | 若 Socket 已采用 TouchSocket，可评估统一框架收益；仍须独立做协议一致性验证 |

维护证据：NModbus 默认分支 [baddfca](https://github.com/NModbus/NModbus/commit/baddfca14e2852373a238aee8d234d6627983b20)，2026-06-30；FluentModbus 默认 dev 分支 [5f45dba](https://github.com/Apollo3zehn/FluentModbus/commit/5f45dbae80d0d5c0eb884e6398dc08a4ffeb1887)，2026-05-19。NuGet 可见 [NModbus 3.0.83](https://www.nuget.org/packages/NModbus/3.0.83)（2026-04-16）及 [FluentModbus 5.3.2](https://www.nuget.org/packages/FluentModbus/5.3.2)（2025-03-28）；不是承诺选定版本。NModbus 另有 4.x 预发布线，不能混同稳定包。

固定项目文件：[NModbus 目标框架](https://github.com/NModbus/NModbus/blob/baddfca14e2852373a238aee8d234d6627983b20/NModbus/NModbus.csproj) 包含 netstandard 与 net6.0；[FluentModbus 项目文件](https://github.com/Apollo3zehn/FluentModbus/blob/5f45dbae80d0d5c0eb884e6398dc08a4ffeb1887/src/FluentModbus/FluentModbus.csproj) 为 netstandard2.0/2.1，引用 System.IO.Ports 5.0.0。选择新 .NET 时要审查实际依赖图和安全更新，不机械保留旧依赖。

## 弦桥落地建议

- 连接实例保存主机、端口、连接超时、响应超时和允许并发上限；设备实例保留 Unit Identifier，尤其连接串口网关时不能忽略站号。
- 按 MBAP 长度取完整消息，并校验事务标识、协议标识、Unit Identifier 和功能响应。一次 TCP Receive 不等于一个完整 Modbus 响应。
- 默认采用保守的单连接串行请求；只有设备手册和实测允许时才增加流水并发，不能因为 TCP 支持多请求就自动开启。
- 合并连续读取时保持用户组周期，不跨有副作用的写操作合并；连续范围错误时不要将整批结果默认为零。
- 断网重连后先恢复连接和状态，再由明确调度发起后续读操作。未知结果的写入不得自动重放。
- 记录逻辑操作 ID 与协议事务 ID；重连后使用新的连接代次，旧响应不得误关联。

[官方应用规范](https://www.modbus.org/file/secure/modbusprotocolspecification.pdf) 说明数据模型和 TCP ADU；[官方规范索引](https://www.modbus.org/modbus-specifications) 同时提供 TCP 实现指南及独立 Modbus Security 规范。本次 TCP 指南下载未成功，未据此声称完成该文档全文核查。

## 安全与范围边界

普通 Modbus TCP 不能因为“使用 TCP”就被认为有认证或加密。Modbus Security 是另一个明确能力，不可把普通库的 TCP 支持写成已支持 TLS 安全规范。建议本地/受控网段运行、限制目标和写入权限；访问安全具体设计由后续方案确认。

## .NET 适配与未覆盖

NModbus 与 FluentModbus 可作为业务驱动依赖；都不自带弦桥所需的 Web 调试、模板版本和同周期组交互。Linux/ARM64 网络路径通常不涉及串口本机驱动，但仍需实测具体运行时、设备和断线行为，不能宣布所有平台已验收。

## 后续验收证据

- 使用受控模拟服务端与真实设备分别验证半包、粘包、事务号错配和协议异常。
- 一个物理网关后多个 Unit Identifier，不串设备、不把路由错误当作合法数据。
- 连续读取中存在无效地址时明确失败范围；32/64 位值的字节序按模板测试。
- 服务器重启、连接半开、响应迟到和写后断网均有可追踪状态。
- 500 ms 组实际请求频率、响应时间与网页刷新时间分别记录。
