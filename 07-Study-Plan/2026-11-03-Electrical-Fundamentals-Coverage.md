# 2026-11-03｜电学基础：电荷、电场、电压、电流与电阻

> 本批内容默认**待学习 / 待验证**。本次只维护知识库，没有进行带电测量、实机接线或 PLC 控制。

## 覆盖范围与落点

| 对话主题 | 整理位置 | 本次补充 |
|---|---|---|
| 电荷与电场 | [电学基础](../02-PLC/Electrical-Fundamentals.md) | 电荷正负、电场定义、电场方向、电场强度与单位 |
| 电场与电压 | [电学基础](../02-PLC/Electrical-Fundamentals.md) | 电势差、E 与 U 的区别；均匀电场 E=|U|/d 的适用前提 |
| 电流与电子运动 | [电学基础](../02-PLC/Electrical-Fundamentals.md) | I=ΔQ/Δt、约定电流方向与电子漂移方向 |
| 电阻与欧姆定律 | [电学基础](../02-PLC/Electrical-Fundamentals.md) | U=IR 的适用条件、理想电阻例题和功率计算 |
| 开路与闭路 | [电学基础](../02-PLC/Electrical-Fundamentals.md) | 解释开路时可以有电压但没有持续负载电流 |
| PLC 工业应用 | [PLC 基础](../02-PLC/PLC-Basics.md) | 增加电学基础入口；强调 24V 是电势差，输入电流与逻辑状态需查模块规格 |
| 原始提问 | [Inbox](../00-Inbox/2026-11-03_电学基础与电场学习记录.md) | 保存本次连续学习的提问范围 |
| 面试与记忆卡 | [P0 面试复习](../05-Interview/P0-Review.md)、[Anki CSV](../06-Anki/PLC.csv) | 新增 6 道口述题、10 张卡（Q111–Q120） |

## 关键概念校准

1. **电场不是电流**：电场描述电荷在空间中受到的力；电流描述电荷通过截面的速率。
2. **电压不是“推动电荷的力”**：电压是两点之间的电势差；电场是对电荷施加力的物理场。
3. **电场与电压不可混为一谈**：电场是空间中的矢量分布，电压是两点间的标量差；E=U/d 只适用于近似均匀电场的相应情形。
4. **金属电子漂移与约定电流方向相反**：电子带负电；约定电流方向按正电荷运动方向定义。
5. **欧姆定律有适用条件**：不能把 U=IR 当成所有元件在所有状态下都成立的关系。
6. **开路仍可能有电压**：电源端子间可以有电势差，断开的负载支路没有持续电流；瞬态和漏电另行考虑。
7. **PLC 输入不是抽象的理想电阻**：输入电流、阈值、公共端和 PNP/NPN 接线必须看具体模块手册。

## 后续练习

1. 不看笔记画出正、负点电荷的电场方向，并解释电场方向的规定。
2. 用口头语言比较电场、电势、电压、电流和电阻。
3. 完成 24 V / 2 kΩ 理想电阻例题，并说明为什么不能把结果直接套用到任意 PLC 输入。
4. 结合一张实际 PLC 输入模块手册，找出额定输入电压、输入电流、ON/OFF 阈值和接线公共端。
5. 复习 Anki Q111–Q120，再用自己的话回答 P0 面试题。

## 资料与边界

- [OpenStax University Physics Volume 2 — Electric Charges and Fields](https://openstax.org/books/university-physics-volume-2/pages/5-introduction)
- [OpenStax University Physics Volume 2 — Electric Potential](https://openstax.org/books/university-physics-volume-2/pages/7-introduction)
- [OpenStax University Physics Volume 2 — Current and Resistance](https://openstax.org/books/university-physics-volume-2/pages/9-introduction)
- [OpenStax University Physics Volume 2 — Direct-Current Circuits](https://openstax.org/books/university-physics-volume-2/pages/10-introduction)

没有核实用户实训台的 PLC 完整订货号、输入电路规格、传感器输出类型或实际接线；没有进行任何带电操作。电气安全必须以现场电气图、模块手册、风险评估和合格人员指导为准。
