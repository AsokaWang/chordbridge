# MT-SICS与称重设备标签化案例借鉴

核查日期：2026-10-24。本文是基于 GitHub 与官方资料的借鉴研究；“建议”不是已确认的详细设计，也不是已实现或实测结果。用户已选择 .NET 方向，具体版本与依赖尚未锁定。

## 结论

称重场景是验证弦桥核心能力的首个合适案例：一次查询获得重量、单位和稳定性，形成多个数据标签；去皮等动作独立调用。GitHub 的 U-CON 样例可学习配置组织，但不是可以直接移植到 .NET 的开源引擎。

## GitHub参考及许可边界

[PTCInc/User-Configurable-Driver-Examples](https://github.com/PTCInc/User-Configurable-Driver-Examples) 默认 main，GitHub API 最近推送 2024-04-25，未归档。包含 [OHAUS Explorer Scale](https://github.com/PTCInc/User-Configurable-Driver-Examples/tree/main/OHAUS%20Explorer%20Scale)、[Mettler IND780](https://github.com/PTCInc/User-Configurable-Driver-Examples/tree/main/Mettler%20IND780) 等 OPF 配置案例。

[license.md](https://github.com/PTCInc/User-Configurable-Driver-Examples/blob/main/license.md) 是 PTC Proprietary Freeware License 并关联 EULA，不是 MIT/Apache 开源许可。README 明确样例不受支持、未完成完整安全与质量审核、无兼容维护保证。本次只读取资料，不下载运行 OPF，不接受协议或复制受限实现。商用 U-CON 产品与免费示例不能混为一谈。

IND780 目录的 [说明](https://github.com/PTCInc/User-Configurable-Driver-Examples/blob/main/Mettler%20IND780/ReadMe_Sample_U-CON_Profiles.txt) 还提示旧配置使用全局缓冲、多通道不推荐；由此可借鉴实例隔离，不能让多台秤共享可变响应缓冲。

## MT-SICS协议依据

[METTLER TOLEDO NewClassic 官方手册](https://www.mt.com/dam/product_organizations/laboratory_weighing/basic_weighing/Products/MT-SICS/MANUALS/en/MT-SICS_NewClassic_en_30014405.pdf) 的 SI 命令说明：立即返回最近内部净重，不等待稳定；响应以 S 开头，S/D 区分稳定与动态，I/+/- 等表示不可执行或超量程。单位取设备配置，不能硬编码 g。不同型号手册可能不同，IND780、OHAUS、NewClassic 不能共用一套未经验证的命令假设。

## 弦桥建议映射

- 模板定义一个完整“查询重量”事务，生成查询字节、识别完整响应、解析字段。
- 响应中的重量、单位、稳定性分别绑定独立 TagId，使用相同结果批次和采样时间。
- 收到动态重量是有效响应但稳定性为 false，不应把通信质量直接标成故障；不可执行/超量程则按协议映射质量，旧值保留原时间。
- “去皮”绑定动作调用，不把向重量标签写0作为替代；动作执行结果与下一次重量读数分别展示。
- 同一查询关联的标签拖入500ms组，只合并等价读取，不能每个字段发一次查询；设置2s则按单独周期组管理。
- 连续输出模式与一次查询模式分开配置，不能把每条主动推送都当成当前请求应答。

## 可复用测试资料设计

保存稳定/动态重量、负值、不同单位、超量程、忙状态、缺字段、半帧、两帧连到、迟到响应等样例。样例记录设备型号、固件/协议手册版本与配置，不只记录一条“成功报文”。预览、运行与回归使用同一解析逻辑。

## 后续验收证据

不连接真实设备先用人工样例验证字段和错误状态，再用获授权的测试设备采集真实帧；两台秤并发不串缓冲；动态值不冒充稳定；动作超时不重发；单位转换保留原值与原单位。当前没有完成任何真实称重设备测试，也未证明现成 GitHub 项目覆盖全部需求。
