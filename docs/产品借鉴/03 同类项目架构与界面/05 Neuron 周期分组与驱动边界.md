# Neuron 周期分组与驱动边界借鉴

核验日期：2026-10-24。Neuron 核心为 C/C++ 生态，非 .NET 组件；作为调度模型和设备组织参考，不作为当前选定底座。

## 结论

**同周期组的概念参考优先级高，直接采用旧开源界面的优先级低。** 其每组统一采集间隔与用户最新纠正很接近；但弦桥还要自动按周期建组、拖入遵从组周期、定时写入和组合调用，不能认为上游已完整解决。

## 维护与许可边界

- [emqx/neuron](https://github.com/emqx/neuron)，默认 `main`；最新返回提交 [ef6b3ff](https://github.com/emqx/neuron/commit/ef6b3ffe03b73e533500e8a6be2c50268e5bb2d8)，2026-09-21。
- 核心及 Modbus/MQTT/eKuiper 插件按 LGPLv3 提供；NeuronEX 为商业发行版，扩展协议及完整 Dashboard 不能按核心开源许可推导。[README](https://github.com/emqx/neuron/blob/main/README.md) / [LICENSE](https://github.com/emqx/neuron/blob/main/LICENSE)
- README 明确：开源 Dashboard 停在 2.6.3，已暂停开发维护。不要将新版本商业文档截图冒充此旧开源 UI 的能力。
- 上游主要给出 Linux 二进制和构建路线；其 ARM/x86 说明不等于弦桥 Windows、ARM64 的兼容承诺。

## 架构和概念依据

实际目录 [src](https://github.com/emqx/neuron/tree/main/src) 包括 adapter、connection、core、parser、persist，协议插件与核心边界可供研究。不要逐行翻译 C 实现进 .NET；只借鉴契约和资源归属。

官方当前 [Groups and Tags](https://docs.emqx.com/en/neuronex/latest/configuration/groups-tags/groups-tags.html) 明确组有采集间隔，同组标签同周期；并提示观察组处理耗时是否接近周期。该链接属于当前 Neuron/NeuronEX 文档体系，不是旧开源 Dashboard 的功能证明。

## 界面资料与建议

上述文档描述驱动节点→组列表→标签列表的配置路径，标签字段包含地址、类型、读写属性和单位，并有添加标签与监视页面图。没有在本次运行其 UI，布局结论限于官方流程，不声称像素检查。

弦桥可保留一个全局“周期视图”：500 ms、1 s、2 s 等分组，显示每组引用数量、最近耗时、迟到、错误和暂停状态；同时设备视图仍可查看标签定义。用户从一个视角操作，另一个视角立即看到同一引用，不产生复制标签。

## 必须与弦桥需求保持的差异

- 上游写属性意味着允许写，不等于按组周期重复写。弦桥的读、写、动作、组合调用要作为明确操作类型配置。
- 同周期不等于可并发访问同一串口；需要按物理连接序列化完整事务。
- 上游分组还涉及上传、订阅目的地；弦桥不能因此强制引入 MQTT 和云端。
- 文档给出的最小间隔、数量上限是上游限制，不直接继承为弦桥指标。

## 落地与验证建议

使用周期组作为计划层、连接执行器作为物理执行层、标签结果作为数据层。重复读取可合并，但写入不可随意合并；超期需要显式统计，不无限补跑。建议以同串口 500 ms 和 2 s 两组混合、慢响应、手动插入和中断迟到响应作为原型验证集。当前尚未实现或验证。
