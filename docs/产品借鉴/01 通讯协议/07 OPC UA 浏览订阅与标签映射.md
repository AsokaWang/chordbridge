# OPC UA 浏览订阅与标签映射借鉴

核查日期：2026-10-24。本文为 GitHub 与官方资料研究，不是代码实现或硬件实测。

.NET 是用户已确认的实现方向；下述具体库和设计仍是候选建议。Windows x64、Linux x64、Linux ARM64、Windows ARM64 必须逐项验证，不能由“.NET Standard”推导出驱动和全部功能已经兼容。

## 结论

优先评估 OPC Foundation 官方 .NET 客户端栈。UA 已有节点、类型、质量和时间语义，弦桥应映射和保留这些信息，而不是把它降为只有数值的“串口式标签”。

## 候选比较

| 项目 | 核实事实 | 弦桥建议 |
| --- | --- | --- |
| [OPCFoundation UA-.NETStandard](https://github.com/OPCFoundation/UA-.NETStandard) | 官方 .NET 参考栈，客户端/服务端等模块；当前 master 为 2.0，master378 维护 1.x | 首选对照验证；新旧分支 API 不可混抄 |
| [Workstation.UaClient](https://github.com/convertersystems/opc-ua-client) | MIT；浏览、读写、订阅；[NuGet 3.2.3](https://www.nuget.org/packages/Workstation.UaClient/3.2.3) 含 net6.0/netstandard2.0 | 较轻的客户端候选，借鉴映射概念；不要照搬 WPF/XAML 界面到 Web 后台 |
| [QuickOPC](https://github.com/OPCLabs/QuickOPC) | 商业组件，有 GitHub 示例；封装较高层 OPC 调用 | 作为商业工期/维护成本对照，不作为免费开源替代品 |

## 版本与许可核查

- 官方栈默认分支固定提交 [f3980f9](https://github.com/OPCFoundation/UA-.NETStandard/commit/f3980f9e72aa779ad43bd0b3c77d9559467f6812)，2026-10-23。该提交 [LICENSE.txt](https://github.com/OPCFoundation/UA-.NETStandard/blob/f3980f9e72aa779ad43bd0b3c77d9559467f6812/LICENSE.txt) 明确是 OPC Foundation MIT License 1.00。不能沿用历史文章的许可证结论，也不能把此结论扩展到所有 OPC Foundation 工具、规范和旧版本。
- [Releases](https://github.com/OPCFoundation/UA-.NETStandard/releases) 本次显示 2.0.1；候选评估需固定实际 NuGet 版本及源提交，不能直接跟随 master 自动升级。
- Workstation 固定提交 [75940cd](https://github.com/convertersystems/opc-ua-client/commit/75940cddb1c67c77adb2a19e91a8423f32833c84)，2025-09-01。维护时间晚于早期文章，但不等于支持所有 UA 功能或平台。

## 弦桥落地建议

1. 浏览树保留命名空间 URI、NodeId、节点类别、类型、读写能力。显示名称/别名不替代节点身份；命名空间索引变化需要重新解析。
2. “查看当前值”读弦桥缓存；“主动读取”发 UA Read；“写入”发 Write；方法调用按支持能力映射动作，不能把全部节点当可写变量。
3. 保留 DataValue 的状态码、源时间和服务器时间，错误/不确定质量不能转换为正常数值。
4. 周期组依然按用户设定统一周期，但 UA 原生订阅的采样间隔、发布间隔和服务端修订值需分别显示。订阅推送不是每 500 ms 必然读硬件一次。[MonitoredItem 规范](https://reference.opcfoundation.org/specs/OPC-10000-4/5.13.1)
5. 对“每周期主动调用读取”和“订阅监视”分清执行模式。若用订阅优化，需要保持产品语义可解释，不能静默替代用户要求的实际读取。
6. 连接、会话、订阅生命周期属于后台；网页刷新不重建全部订阅，不重放控制指令。
7. 证书信任、服务端身份、用户凭据和权限错误要可诊断。演示程序的匿名或无加密设置不是生产默认值。

## 未覆盖与平台风险

官方库的测试不等于弦桥已认证，更不等于设备兼容性已通过。Windows/Linux x64/ARM64 都应验证证书存储、加密依赖、断线恢复和实际服务器；Native AOT 应单独验证，不从 README 声明推导全部插件和脚本均可 AOT。

## 后续验收证据

- 浏览大型树及取消浏览，建立多个标签后改别名，不丢失 NodeId 关联。
- 测试服务端修订采样率、坏质量、无权限写入、类型不匹配和证书不信任。
- 断网与服务端重启后恢复监视，不重复写入；源时间与页面时间清晰区分。
- 使用实际 OPC UA 服务器记录 Read/Write/订阅证据，而不是仅运行库自带样例。
