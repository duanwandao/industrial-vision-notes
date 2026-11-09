# 2026-11-09 WinForms 布局与数据绑定覆盖报告

## 本批新增

- [WinForms 布局与数据绑定](../01-CSharp/WinForms-Layout-and-Data-Binding.md)：TableLayoutPanel、FlowLayoutPanel、Panel、DataGridView、DialogResult、ShowDialog Owner、using/Dispose、模式匹配、BeginUpdate/EndUpdate、BindingList<T>、INotifyPropertyChanged。
- [课堂原始记录](../00-Inbox/2026-11-09-WinForms-Layout-Dialogs-DataBinding.md)。
- C# 索引、学习进度、P0 面试复习与 Anki 同步更新。

## 重点理解

1. TableLayoutPanel 负责行列网格；FlowLayoutPanel 负责流式排列；Panel 是普通容器。
2. DataGridView 可使用非绑定模式，也可通过 DataSource 绑定对象集合；真实业务记录通常把模型作为数据来源。
3. DialogResult 是对话框返回值；ShowDialog(this) 指定 Owner；using var 在作用域结束时释放临时窗体。
4. Form2 通过 CreatedLight 返回 Light；`is not Light light` 只做类型匹配并声明引用变量，不会创建新 Light。
5. BeginUpdate/EndUpdate 是控件重绘优化，不是线程同步工具。
6. BindingList<T> 通知集合变化；INotifyPropertyChanged 通知条目属性变化。计算属性 DisplayText 依赖其他属性时也要发出 DisplayText 通知。

## 验证边界

用户已提供设备管理主窗体布局截图，说明设计器布局已初步完成。尚未提供完整 Form2/Form1 代码和运行验证，因此：
- 不宣称 DialogResult 数据回传、地址重复校验、BindingList 自动刷新已在用户项目中运行成功。
- 未验证 ListBox 在目标项目中对 DisplayMember + INotifyPropertyChanged 的具体刷新效果。
- 未连接 PLC、灯控器或真实设备；Q0.x 地址仅为示例格式。
- 本批为 .NET WinForms 教学内容，不替代 PLC 硬件输出反馈与安全控制。

## 后续实践

1. 完成 Form2 模态参数窗口和 CreatedLight 返回。
2. 使用 BindingList<Light> 绑定 ListBox，验证 Add/Remove/Clear。
3. 实现 INotifyPropertyChanged，修改 IsOn/Name，观察 DisplayText 更新。
4. 完成编辑/删除/清空与选中项状态显示。
5. 后续把 DataGridView 用于检测记录列表，并在真实相机回调场景中练习 UI 线程调度。
