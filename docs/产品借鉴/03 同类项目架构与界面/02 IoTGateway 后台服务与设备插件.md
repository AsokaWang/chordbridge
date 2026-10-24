# IoTGateway 后台服务与设备插件借鉴

核验日期：2026-10-24。本文特指 `iioter/iotgateway`，不是同名的 PeterWaher/IoTGateway。研究状态：阅读仓库与部分源码，未启动或设备实测。

## 结论

**优先级高，是 .NET 后台加网页、设备和变量管理的直接参考。** 适合阅读如何把设备生命周期接到后台宿主、驱动接口和 Web 配置，但它的南向采集、北向平台转发中心不等于弦桥的测试工作台。

## 项目与授权边界

- [GitHub 仓库](https://github.com/iioter/iotgateway)，默认 `master`；最新返回提交 [8de442a](https://github.com/iioter/iotgateway/commit/8de442a7dec52d2993d1e9667d799846936d681b)，2026-06-21，升级到 .NET 10。旧文章中 .NET 6/8 不能作为当前版本依据。
- [LICENSE](https://github.com/iioter/iotgateway/blob/master/LICENSE) 为 MIT。项目另有企业版；MIT 仓库与企业功能、第三方驱动授权分开核查。
- README 提到 OPC UA 授权注意事项，不能据此笼统判定所有 OPC UA 必须收费，也不能忽略实际 SDK、证书和分发条款。应针对最终依赖版本单独审核。[项目说明](https://github.com/iioter/iotgateway/blob/master/README.md)

## 真实源码入口和观察

- [IoTBackgroundService.cs](https://github.com/iioter/iotgateway/blob/master/Plugins/Plugin/IoTBackgroundService.cs)：继承 BackgroundService，通过宿主启动通知取得 DeviceService 和 ModbusSlaveService。该文件的 ExecuteAsync 和 StopAsync 返回已完成任务，**不是可直接抄作完整优雅停止范例**，资源清理须继续追踪其他实现。
- [DeviceThread.cs](https://github.com/iioter/iotgateway/blob/master/Plugins/Plugin/DeviceThread.cs)：观察到 CancellationTokenSource、暂停事件、驱动实例及方法委托缓存；启动时把变量状态设为 Bad。此处展示“设备运行对象”承担状态和执行职责的做法，未证明满足弦桥的组合事务及中断语义。
- [PluginInterface](https://github.com/iioter/iotgateway/tree/master/Plugins/PluginInterface)、[Drivers](https://github.com/iioter/iotgateway/tree/master/Plugins/Drivers)：协议契约与驱动实现分开。
- [Model](https://github.com/iioter/iotgateway/tree/master/IoTGateway.Model)、[DataAccess](https://github.com/iioter/iotgateway/tree/master/IoTGateway.DataAccess)、[ViewModel](https://github.com/iioter/iotgateway/tree/master/IoTGateway.ViewModel)：模型、数据访问与页面逻辑分层的阅读入口。

## 界面资料及可借鉴布局

已下载并实际查看官方仓库的 [写入值弹窗截图](https://github.com/iioter/iotgateway/blob/master/images/set-variable.png)（752×460，核验日期 2026-10-24；文件名为 set-variable.png，但画面是“写入值”，不是完整变量配置页）。可直接观察到：

- 顶部标题“写入值”，主体按纵向分块显示标签路径 Modbus/outQty、当前原始值和计算值、数据类型 int16 与质量状态 Good。
- 下方单独提供“设定原始值”输入框，右下角是“下发”和“取消”。这把当前状态与将要写入的值分开，避免把输入值误当作当前读数。
- 这些是静态截图观察，不证明下发后的确认、回读、权限和超时行为。弦桥可借鉴信息次序，但把操作入口接到统一标签调用，不能据此改成按钮中心的产品。

另有 [表达式图](https://github.com/iioter/iotgateway/blob/master/images/express.png) 和 [变量动态图](https://github.com/iioter/iotgateway/blob/master/images/variables.gif)，本次只验证文件存在，未查看画面。官网教程抓取超时；未启动项目或做交互验收。

弦桥建议以设备为上下文：选择设备后查看标签配置和实时表，旁边按需展开原始报文与解析。配置表与实时结果表要有明显模式区别，避免用户误以为修改名称就会立即发送指令。别名只用于展示，不用于覆盖标签稳定 ID。

## 弦桥适配重点

1. 学习驱动接口和参数描述，便于一个网页编辑器适配不同连接类型。
2. 后台运行对象负责连接和任务生命周期，页面只是调用者；不要因浏览器关闭而释放串口。
3. 将底层读取、写入、动作映射为统一标签调用，保留请求与反馈的不同含义。
4. 设备执行器之上再建立统一周期组；不能把上游按别名分组上传当作用户要求的 500 毫秒定时组。
5. 把暂停、取消和释放资源设计成可验证状态机，不能把取消令牌触发等同于设备动作撤回。

## 不照搬和验收建议

不自动加入 MQTT 云平台、3D 数字孪生、数据库归档和内置协议服务器。它们是上游能力，不是当前需求。

后续验证设备禁用、后台停止、异常重连、配置变更以及串口迟到响应；检查所有任务和连接能否受控释放。对于串口流程，不允许多个 Web 请求直接并发调用驱动。上游宣称跨平台仍需弦桥逐架构实测。
