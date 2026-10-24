# Modbus RTU 寄存器与事务借鉴

核查日期：2026-10-24。本文为 GitHub 与官方资料研究，不是代码实现或硬件实测。

.NET 是用户已确认的实现方向；下述具体库和设计仍是候选建议。Windows x64、Linux x64、Linux ARM64、Windows ARM64 必须逐项验证，不能由“.NET Standard”推导出驱动和全部功能已经兼容。

## 结论

优先复用专用 Modbus 库，不自行重写 CRC 与功能码。弦桥应补的是可理解的寄存器配置、标签映射、设备模板和总线调度，而非再造协议栈。

## GitHub 候选比较

| 项目 | 已核实能力和许可 | 适用判断 |
| --- | --- | --- |
| [NModbus](https://github.com/NModbus/NModbus) | MIT；RTU、ASCII、TCP、UDP；串口适配有独立项目 | 优先候选，接口层清晰，可将传输与协议隔开；不要选到已沉寂的 NModbus4 原分支 |
| [FluentModbus](https://github.com/Apollo3zehn/FluentModbus) | MIT；TCP/RTU 客户端和服务端，常用功能码与部分扩展 | 轻量对照候选，特别适合搭建受控模拟端；必须核对目标设备所需功能码 |
| [TouchSocket.Modbus](https://github.com/RRQM/TouchSocket) | 当前公开 README 列 RTU/ASCII/TCP 主站模块 | 若 Socket 已采用 TouchSocket，可评估统一框架收益；仍须独立做协议一致性验证 |

维护证据：NModbus 默认分支 [baddfca](https://github.com/NModbus/NModbus/commit/baddfca14e2852373a238aee8d234d6627983b20)，2026-06-30；FluentModbus 默认 dev 分支 [5f45dba](https://github.com/Apollo3zehn/FluentModbus/commit/5f45dbae80d0d5c0eb884e6398dc08a4ffeb1887)，2026-05-19。NuGet 可见 [NModbus 3.0.83](https://www.nuget.org/packages/NModbus/3.0.83)（2026-04-16）及 [FluentModbus 5.3.2](https://www.nuget.org/packages/FluentModbus/5.3.2)（2025-03-28）；不是承诺选定版本。NModbus 另有 4.x 预发布线，不能混同稳定包。

固定项目文件：[NModbus 目标框架](https://github.com/NModbus/NModbus/blob/baddfca14e2852373a238aee8d234d6627983b20/NModbus/NModbus.csproj) 包含 netstandard 与 net6.0；[FluentModbus 项目文件](https://github.com/Apollo3zehn/FluentModbus/blob/5f45dbae80d0d5c0eb884e6398dc08a4ffeb1887/src/FluentModbus/FluentModbus.csproj) 为 netstandard2.0/2.1，引用 System.IO.Ports 5.0.0。选择新 .NET 时要审查实际依赖图和安全更新，不机械保留旧依赖。

## 协议不能省略的约束

RTU 使用地址、功能码和 CRC，按串行规范识别帧间隔；一个主站在总线上一次发起一个事务。广播地址 0 不返回响应，不能按正常请求等待并自动重试。[官方串行规范 V1.02](https://www.modbus.org/file/secure/modbusoverserial.pdf)；不要以标注“legacy only”的旧串行规范作为新实现依据。[规范入口](https://www.modbus.org/modbus-specifications)

寄存器地址、位区和功能码是协议数据，不等于业务名称。读保持寄存器、输入寄存器、线圈和离散输入应明确区分；设备手册的 40001 风格编号与协议偏移不能混用。[应用规范 V1.1b3](https://www.modbus.org/file/secure/modbusprotocolspecification.pdf)

## 弦桥落地建议

- 标签保存站号、存储区、偏移、数量、类型、字序、缩放、单位及读写权限；别名与地址分离。
- 原始寄存器值与换算后的工程值同时可检查；浮点和多寄存器字序按设备模板指定，不假设每台设备相同。
- 批量读取只合并同一设备、兼容功能区、兼容周期且没有非法空洞的连续范围。设备最大读取量小于标准上限时遵从设备限制。
- 一个 500 ms 组可包含多台设备，但最终经物理串口队列顺序执行；将超期和排队耗时反馈到组，不无限补跑。
- 先写后读作为连接独占的完整组合流程；普通写响应只表示协议接受情况，读回相等也不自动证明物理动作完成。
- 默认不进行全地址扫描或广播写入。诊断功能必须明确目标与范围，不能为了自动发现而扰动设备。

## 未覆盖与平台风险

协议库不会自动发现寄存器的业务含义，也不能解决 RS485 电气故障。RTU 对时序敏感，USB 驱动缓冲及普通操作系统调度不能当作硬实时保证。NModbus 的具体串口适配器版本和 RJCP 本机库需要随目标架构验证。

## 后续验收证据

- 对照设备手册验证地址偏移、读写功能码、CRC、异常响应和字序。
- 两台不同站号设备共用串口，500 ms 与 2 s 组混合提交，不发生事务交叉。
- 延迟/错误站号/错误 CRC 的响应不更新当前标签；写超时不自动重复。
- 设备与串口适配器真实测试，含断线恢复和超过周期的情况；同时保留报文与时间证据。
- 若支持广播，单列其无响应语义及用户明确操作边界；否则明确不支持，而不是误报超时。
