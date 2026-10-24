# ThingsGateway 网关分层与调试管理借鉴

核验日期：2026-10-24。定位：.NET 工业网关参考，不是弦桥已选底座。本文为资料研究，未启动项目、未进行设备或跨平台测试。

## 结论

**优先级高。最值得借鉴的是通道、设备、变量分层，以及应用逻辑与 Web 管理界面的分离。** 它比完整 SCADA 更贴近弦桥的 .NET 后台和浏览器目标，但仍以采集和转发为中心，不能直接替代弦桥的交互式通讯测试、统一动作标签和串口事务调度。

## 已核实的项目事实

- 仓库：[ThingsGateway/ThingsGateway](https://github.com/ThingsGateway/ThingsGateway)，本次默认分支为 `v12`。
- 维护证据：默认分支最新返回提交为 [b9c92a5](https://github.com/ThingsGateway/ThingsGateway/commit/b9c92a5ca22913a074e5d3f01adec88c7fd7389a)，2026-09-14，依赖更新。这是维护活动证据，不是稳定性认证。
- 根目录为 Apache-2.0；README 同时介绍 PRO 插件和另一套 runtime 专业版。不能把专业版宣传、所有协议列表、源码可见性混同为开源核心可直接复用。复制前逐项核对目标文件、插件包、依赖许可证。[许可证](https://github.com/ThingsGateway/ThingsGateway/blob/v12/LICENSE) / [README](https://github.com/ThingsGateway/ThingsGateway/blob/v12/README.md)
- README 记载 .NET 8/10、Windows/Linux 及 x64/ARM64 镜像。这里只记录上游声明，不能据此宣布弦桥已兼容各平台。

## 真实架构入口

| 已查看目录 | 阅读价值 |
| --- | --- |
| [Gateway.Application](https://github.com/ThingsGateway/ThingsGateway/tree/v12/src/ThingsGateway.Gateway.Application) | 存在 Driver、Entity、Services、Job、Logging、CSScript 等职责目录，适合研究业务与基础设施如何分开 |
| [Gateway.Razor](https://github.com/ThingsGateway/ThingsGateway/tree/v12/src/ThingsGateway.Gateway.Razor) | 页面与组件单独组织，不把设备 I/O 全写入界面 |
| [Pages](https://github.com/ThingsGateway/ThingsGateway/tree/v12/src/ThingsGateway.Gateway.Razor/Pages) | 实际有 GatewayMonitorPage、LogPage、PluginPage、Management 等入口 |
| [Server](https://github.com/ThingsGateway/ThingsGateway/tree/v12/src/ThingsGateway.Server) | Web 宿主，与界面库、应用库分离 |

以上是目录与官方说明核验，不等于已经审计调度、重试和脚本沙箱的具体实现。

## 界面布局借鉴

本次未启动 UI，也未像素检查截图。以下是根据页面目录与官方功能说明提出的弦桥布局建议，而非对其页面坐标的复刻：

1. 左侧以连接、设备、标签为导航，顶层另列模板、定时组和主机状态。
2. 中间保留高密度标签表，显示别名、类型、当前值、质量、更新时间、所属调用与周期。
3. 设备详情将连接设置、原始报文、解析预览和执行记录分 Tab；不要把所有运维功能挤入首页。
4. 主机状态与设备通讯状态分开：主机 CPU 正常不等于设备连接成功。

## 如何落到弦桥

建议采用其“分层和插件契约”思想，不直接搬完整网关。弦桥可先形成连接管理、协议适配、模板与标签、调用调度、Web 展示几个清晰边界；模块名称及实现框架仍待方案确定。

- 设备模板保存通用规则，实例保存地址和覆盖参数；现有上游设备配置模型不自动满足模板版本差异确认。
- 批量读取只作为物理请求优化，同周期组与标签调用结果由弦桥自己的契约控制。
- 只允许协议适配层持有串口，页面、脚本和定时组通过同一调用入口执行。
- C# 脚本功能存在不表示安全隔离已经解决；必须另行验证权限、资源限制及异常退出。

## 不直接照搬及验证门槛

不引入未经要求的云转发、规则引擎、告警或商业插件。不把断线自动重连转化为写命令自动重发。

后续原型至少验证：关闭网页后台仍运行；同一串口手动和定时不争用；模板升级不修改未选择的实例；页面旧值明确标旧；插件故障可定位；目标四种系统架构逐项验证。当前均未实测。
