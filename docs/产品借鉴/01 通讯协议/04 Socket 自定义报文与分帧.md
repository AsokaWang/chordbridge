# Socket 自定义报文与分帧借鉴

核查日期：2026-10-24。本文为 GitHub 与官方资料研究，不是代码实现或硬件实测。

.NET 是用户已确认的实现方向；下述具体库和设计仍是候选建议。Windows x64、Linux x64、Linux ARM64、Windows ARM64 必须逐项验证，不能由“.NET Standard”推导出驱动和全部功能已经兼容。

## 结论

建议把 Socket 视为传输能力，设备协议解析保持独立。TCP 是字节流，必须分帧；UDP 是数据报，但仍需校验、来源匹配与丢包处理。当前需求只确认 Socket，TCP/UDP 与客户端/服务端的交付范围仍应逐项确定。

## 候选比较

| 项目 | 已核实事实 | 弦桥采用建议 |
| --- | --- | --- |
| [TouchSocket](https://github.com/RRQM/TouchSocket) | Apache-2.0；数据适配器、插件和串口/网络统一接口；TouchSocketPro 商业线另计 | 优先验证自定义协议分帧与诊断能力，避免一次接入全部生态 |
| [SuperSocket](https://github.com/kerryjiang/SuperSocket) | Apache-2.0；PipelineFilter、会话、命令分发及客户端组件 | 若需要设备主动连入或模拟服务端，重点比较其会话/管道设计 |
| [dotnet/runtime 的 System.Net.Sockets](https://github.com/dotnet/runtime/tree/main/src/libraries/System.Net.Sockets) | 官方基础实现，MIT | 作为低依赖基线；要自行补分帧、重连、诊断和关联逻辑 |

固定证据：TouchSocket [30a3bb2](https://github.com/RRQM/TouchSocket/commit/30a3bb25e89ef5706e4e47098b5a76a52ab151a7)，2026-10-11，版本文件 4.3.10；SuperSocket [58406a9](https://github.com/kerryjiang/SuperSocket/commit/58406a98b7a8b771b99e747ad954ce1e53b129b2)，2026-10-03。[SuperSocket Releases](https://github.com/kerryjiang/SuperSocket/releases) 本次可见 2.1.0。固定提交与发布包不是同一概念，选型后还需锁定实际包版本。

## 弦桥落地建议

- `Transport → FrameDecoder → TransactionMatcher → TagMapper` 四层分开。接口名为建议，不是已实现 API。
- 支持终止符、固定长度、长度头三类可视化规则，再允许脚本处理设备私有格式。
- 帧长度、接收缓冲、处理时间均设置上限，避免无结束符或错误长度导致后台内存增长。
- 有协议事务号就校验事务号；没有时连接内串行，绝不能把“下一批字节”默认当成新查询结果。
- 连接重建与命令重发分开：重连成功不代表之前写命令未执行，更不能重放写命令。
- 同一条设备响应映射多个标签，但有副作用的写入和动作不做去重合并。
- 网页实时通信与设备 Socket 是不同链路；关闭网页不能关闭设备接收循环。

## 可复用程度与边界

可复用网络传输、分帧管道、连接状态通知；库不会提供弦桥的标签模板和同周期调度规则。README 的性能数字是作者测试场景，不是弦桥吞吐承诺。用户脚本和模板不得自行绕过连接调度直接抢占 Socket。

## 后续验收证据

- 模拟半包、粘包、断连、空帧、超长帧、错事务号和迟到响应。
- 对 UDP 单独验证乱序、重复、丢失及非目标来源，不假设传输可靠。
- 同周期任务超期不重叠；手动中断不污染后续结果。
- 目标平台分别完成实际设备/受控模拟端到端验证，记录延迟与内存上限。

本次未下载或执行上述仓库程序，未连接生产设备。
