# Rapid SCADA 运行分层与标签表借鉴

核验日期：2026-10-24。研究对象为 Rapid SCADA 6；仅资料与源码结构阅读，未运行项目。

## 结论

**优先级中高：适合借鉴后台通讯与 Web 展示分离、表格看数与诊断分开，不适合直接搬完整 SCADA。** 弦桥更偏工程调试，不需要一开始引入归档服务器、分布式冗余和大规模组态系统。

## 项目信息和维护

- [RapidScada/scada-v6](https://github.com/RapidScada/scada-v6)，默认 `master`。
- 最新返回提交 [7c11977](https://github.com/RapidScada/scada-v6/commit/7c11977fb21b3445591c939972115fd613db7681)，2026-09-07，合并开发分支。
- 根仓库 [Apache-2.0 许可证](https://github.com/RapidScada/scada-v6/blob/master/LICENSE.txt)。官方网站另有模块商店和企业版，不能认为所有图表、报表、驱动和运维模块都随根仓库免费提供。购买或复制扩展前核查具体模块条款。[官方文档](https://rapidscada.org/product/documentation/)

## 架构事实及代码位置

官方明确区分 Communicator、Server、Webstation：通讯器接设备并做诊断，Server 处理归档和计算，Webstation 提供浏览器表格、图形和报表；组件通过 TCP 交互，可部署于同机或分布式。[架构说明与图](https://rapidscada.net/docs/en/latest/software-overview/architecture)

阅读入口：[ScadaComm](https://github.com/RapidScada/scada-v6/tree/master/ScadaComm)、[OpenDrivers](https://github.com/RapidScada/scada-v6/tree/master/ScadaComm/OpenDrivers)、[ScadaServer](https://github.com/RapidScada/scada-v6/tree/master/ScadaServer)、[ScadaWeb](https://github.com/RapidScada/scada-v6/tree/master/ScadaWeb)、[ScadaCommon](https://github.com/RapidScada/scada-v6/tree/master/ScadaCommon)。目录分离本身不能证明模块运行可靠，具体实现仍需按版本审阅。

## 界面资料和借鉴

官方 [Table Views 文档](https://rapidscada.net/docs/en/latest/configuration/table-views) 描述左侧按设备分组的通道树，主区域为表格编辑和查看；查看表格条目可转入趋势，命令另走对话流程。

- [官方表格截图](https://rapidscada.net/docs/en/latest/configuration/table-views-files/table-view-en.png)
- [官方表格编辑器截图](https://rapidscada.net/docs/en/latest/configuration/table-views-files/table-editor-en.png)

本次核实文档与图片链接，未进行截图像素验收或实际交互。弦桥可借鉴“左侧设备标签树、中间实时表、按需展开趋势/调用详情”，但主入口仍是标签调用，不变成按钮组态产品。实时值、质量、采样时间和调用进度应同时可见。

## 落地建议

保留职责边界，不必保留进程数量。初版可在一个 .NET 后台内实现相互独立的通讯、标签、调度、网页模块；只有 OPC DA 平台桥接、故障隔离或部署需求成立后才考虑拆进程。

将配置模型与运行快照分开：编辑模板不直接改正在运行的事务；读取缓存不触发设备通讯；控制结果不得仅根据网页提交成功标成功。

## 不照搬和验证门槛

不引入本次未要求的日报、告警、无限历史存储、分布式冗余。上游表格周期可能是历史展示间隔，不等于弦桥设备触发周期。

验证应覆盖浏览器刷新不中断后台；一条标签错误不拖垮全表；列表滚动不发设备请求；控制待确认和结果未知可追踪；后台退出时事务状态不会被伪装成功。当前只有参考研究，未做这些测试。
