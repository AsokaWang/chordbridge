# FUXA Web 设备标签与视图分离借鉴

核验日期：2026-10-24。FUXA 为 Node.js 加 Angular 项目，**不是 .NET 库**；本篇主要用于产品结构和 Web 交互参考，未运行项目。

## 结论

**界面设计参考优先级高，直接技术底座优先级低。** 借鉴设备连接、标签定义与展示视图的分离，以及在同一浏览器内编辑和监视的思路；不把弦桥扩展成 SVG 工艺组态工具。

## 项目与维护证据

- [frangoteam/FUXA](https://github.com/frangoteam/FUXA)，默认 `master`。
- 本次最新返回提交 [874a68a](https://github.com/frangoteam/FUXA/commit/874a68a59c39baa72fb098e97280d1dcd82b83bf)，2026-10-21，修复 S7 参数类型处理。
- 仓库为 MIT；依赖及插件须分别核查，尤其不能把某种协议名称出现在文档当作已验证的 ARM64 驱动能力。[仓库许可证入口](https://github.com/frangoteam/FUXA)
- 官方说明后台 Node.js、前端 Angular/HTML/SVG，支持多种工业协议与 Web 工程编辑。[README](https://github.com/frangoteam/FUXA/blob/master/README.md)

## 架构阅读入口

- [server/runtime](https://github.com/frangoteam/FUXA/tree/master/server/runtime)：实际有 devices、project、scripts、scheduler、storage 等模块。
- [server/api](https://github.com/frangoteam/FUXA/tree/master/server/api)：HTTP 接口入口。
- [server/runtime/devices](https://github.com/frangoteam/FUXA/tree/master/server/runtime/devices)：设备层；阅读时重点追踪读写结果如何进入实时标签。
- [server/runtime/project](https://github.com/frangoteam/FUXA/tree/master/server/runtime/project)：项目配置边界，适合研究“配置与运行时”如何衔接。

以上目录已查看，未逐函数审计。不能据目录名称断言实现了事务取消或可靠写入恢复。

## 界面依据

官方 [Devices and Tags](https://frangoteam.github.io/FUXA/HowTo-Devices-and-Tags/) 从编辑器 Connections 进入设备与标签；OPC UA 连接建立后添加标签；标签可配置类型和读写转换脚本。[连接设置图片](https://frangoteam.github.io/FUXA/images/setup-connections.png)

已下载并实际查看上述官方连接设置截图（375×308，核验日期 2026-10-24）。可观察到浏览器地址为 localhost:1881/editor；左侧为 Views/MainView 和 General、Controls 图元工具栏；顶部齿轮打开右侧覆盖式 Setup 面板，面板以图标网格提供 Views、Line charts、Connections、Users，Connections 被红框突出。截图只显示设置入口，不显示连接参数、标签列表或收发报文。

可借鉴点是“主工作区保持上下文，设置入口按职能分类”。不宜照搬的是把 Connections 隐藏在组态编辑器的齿轮里：弦桥以通讯测试为主，连接和标签应是一级工作区，而不是画布的附属设置。这是基于画面的设计建议，不是 FUXA 缺陷判断。

[UI Layout](https://frangoteam.github.io/FUXA/HowTo-UI-Layout/) 文字另外区分起始视图、导航侧栏、顶部栏及编辑/运行入口；这些额外功能仅作文档依据。未启动项目、未交互验收，也未验证其他截图或 GIF；部分 GIF 抓取不受当前工具支持。

## 对弦桥的具体建议

1. 连接和标签是业务对象，趋势或视图只引用稳定 ID；删视图不删标签。
2. 标签详情里同时放读配置、写配置和转换预览，明确“修改定义”与“立即执行”。
3. 顶层使用连接测试、标签、模板、定时组、主机状态几个工作区，而不是把组态画布作为首页。
4. 展示更新可以合批，但不能丢失执行结果；大量原始报文应有独立滚动区和暂停显示功能，暂停显示不能暂停后台通讯。

这些是本项目的设计推导，不代表 FUXA 已具备弦桥所有语义。

## 需避免的迁移错误

不能把上游 Scheduler 直接等同于 500 毫秒统一周期组；不能默认脚本编辑器就是安全沙箱；不能把读写转换静默失败当作有效标签值。FUXA 说明中的脚本参数、注释限制属于其特定实现，弦桥无需沿用限制。

后续验收建议：标签改名不破坏引用；看缓存不新增读取；重开网页恢复快照；模板配置修改可回归样例；控制超时明确未知。本次未测试。
