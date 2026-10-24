# OPC DA Windows 兼容与桥接借鉴

核查日期：2026-10-24。本文为 GitHub 与官方资料研究，不是代码实现或硬件实测。

.NET 是用户已确认的实现方向；下述具体库和设计仍是候选建议。Windows x64、Linux x64、Linux ARM64、Windows ARM64 必须逐项验证，不能由“.NET Standard”推导出驱动和全部功能已经兼容。

## 结论

OPC DA 是必须保留的需求，但不能承诺 Linux 原生 COM/DCOM 接入。建议把 DA 接入隔离为 Windows 适配进程，再评估与跨平台后台的桥接；这只是候选方案，不能替用户默认确认部署边界。

[OPC Foundation 官方说明](https://opcfoundation.org/about/opc-technologies/opc-classic/) 将 OPC Classic 定义为基于 Windows COM/DCOM 的体系。Web 页面能在 Linux 打开，不表示 DA 驱动也能在 Linux 原生运行。

## GitHub 候选比较

| 项目 | 证据 | 适用判断 |
| --- | --- | --- |
| [TitaniumAS.Opc.Client](https://github.com/titanium-as/TitaniumAS.Opc.Client) | MIT；DA 1.0/2.05A/3.0，浏览、异步读写和变化订阅；COM 包装 | 可读源码与接口设计，旧 Windows 接入实验候选；不应直接作为现代跨平台 .NET 核心依赖 |
| [OPCLabs QuickOPC](https://github.com/OPCLabs/QuickOPC) 与 [C# 示例](https://github.com/OPCLabs/Examples-ConnectivityStudio-CSharp) | 商业 SDK，有可查看示例；DA、UA 等多协议封装 | 商业支持对照方案，须核对具体 DA 平台/架构支持、版本与交付许可；示例公开不代表 SDK 免费 |

Titanium 默认分支固定提交 [77e34b6](https://github.com/titanium-as/TitaniumAS.Opc.Client/commit/77e34b6cea1b083294042ffd69b10696733d366f)，2022-07-14；[NuGet 1.0.2](https://www.nuget.org/packages/TitaniumAS.Opc.Client/1.0.2) 发布于 2016-04-20，目标 .NET Framework 4.0。仓库与包发布时间不同，不能把源码改动当成包已更新。[固定许可证](https://github.com/titanium-as/TitaniumAS.Opc.Client/blob/77e34b6cea1b083294042ffd69b10696733d366f/LICENSE)

QuickOPC 仓库核验提交 [60e3abd](https://github.com/OPCLabs/QuickOPC/commit/60e3abd66ae71ad511b983ac2359331dadd7f1e7)，2026-02-10。仓库是产品/用法入口，不是完整 SDK 开源实现。其文档说明无许可证时运行试用模式，有时间限制，正式使用需评估商业许可；本次不采购、不安装。[官方授权类型](https://opclabs.com/purchase/license-types)

## 弦桥落地建议

- DA 适配器负责服务器发现、ProgID/CLSID、ItemID、Group 和订阅；主程序只接收统一标签结果与执行状态。
- 别名独立于 ItemID；缓存值和设备读、同步/异步调用的真实能力分别标记。
- 保留 Value、Quality、Timestamp，并将单项 HRESULT/读写结果显示到对应标签，不把一个 Group 成功等同于全部 Item 成功。
- 用户 500 ms 定时组与 DA Group 的请求/修订更新率区分。若服务器调整实际速率应显示，不制造“精确500ms”假象。
- 同步阻塞、COM apartment、回调释放和异常退出应在适配进程隔离，不能阻塞 Web 请求线程。
- 桥接消息需要操作 ID、超时和结果未知状态；桥接连接重建不得重放写入或控制。

Titanium README 明确初始化涉及 MTA 与 COM 安全设置；这些是其接口使用前提，不是可以自动改用户系统权限的授权。[原始使用说明](https://github.com/titanium-as/TitaniumAS.Opc.Client#bootstrapping-the-library)

## 平台与权限风险

- 64 位操作系统上仍可能存在旧 32 位 OPC 组件。用户不支持 32 位操作系统，不等于已经同意引入 32 位辅助进程；须单独评估并确认。
- Windows ARM64 必须验证供应商服务器、COM 代理和运行时架构；不能由“Windows 支持”推断 ARM64 可用。
- 远程 DCOM 权限、认证、Windows 更新后的安全兼容等需与现场 IT 对齐。不能以关闭防火墙或降低安全级别作为默认解决方式。
- Linux 通过 Windows 桥接只是系统级接入方式，不应写成 Linux 原生 DA 已实现。

## 后续验收证据

- 本地与远程实际 DA 服务器分别验证浏览、读、写、订阅、质量与时间。
- 服务端退出、COM 调用卡住、权限不足和桥接断线时，其他协议与网页仍可运行。
- 明确进程/组件位数并记录兼容矩阵；支持失败时提供真实限制，不显示假成功。
- 写入超时保留未知结果，读回核验不自动证明机械动作已完成。

本次只做资料研究，未连接 DA 服务器、修改 DCOM 权限或执行任何系统配置。
