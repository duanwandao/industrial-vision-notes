# 2026-11-09 WinForms 布局、DataGridView、模态窗体与数据绑定课堂记录

> 原始记录摘要；保留用户实际提问与练习上下文。正式整理见 [WinForms 布局与数据绑定](../01-CSharp/WinForms-Layout-and-Data-Binding.md)。

## 用户本轮学习内容与问题

- 学习 TableLayoutPanel：行列网格、ColumnStyles/RowStyles、Absolute/Percent/AutoSize、嵌套布局、SetColumnSpan/SetRowSpan。
- 学习 FlowLayoutPanel：FlowDirection、WrapContents、AutoScroll、Padding、Margin、AutoSize、SetFlowBreak；用它实现工业视觉软件工具栏。
- 学习 DataGridView：列/行/单元格、Columns/Rows、DataSource、DataPropertyName、nameof、CellFormatting、检测记录模型。
- 学习窗体布局：使用 TableLayoutPanel、GroupBox、FlowLayoutPanel、ListBox、Label、Button 组合设备管理界面。
- 用户希望“添加灯”使用独立参数窗口 Form2，而不是在主窗体直接录入。Form2 包含设备名称、PLC 点位、确定/取消按钮。
- 询问 DialogResult.OK / Cancel、using var dialog、ShowDialog(this) 的 Owner 语义、btnAdd_Click 逐步执行过程。
- 询问 BeginUpdate/EndUpdate，以及 `dialog.CreatedLight is not Light light` 是否创建新对象。
- 继续学习 BindingList<T> 和 INotifyPropertyChanged。

## 本轮设计结论

- Form1 负责设备集合和管理；Form2 负责录入/校验参数并返回 Light 对象。
- Form2 使用 `CreatedLight` 属性返回数据；调用方通过 `ShowDialog(this)` 的 DialogResult 判断是否接受。
- `using var` 在作用域结束时 Dispose 临时窗体；不会 Dispose 从窗体取出的业务对象。
- `is not Light light` 是类型模式匹配，声明局部变量并引用已有对象，不执行 new。
- BeginUpdate/EndUpdate 仅控制具体控件的重绘，不是线程锁或线程切换。
- BindingList<T> 通知集合变化；INotifyPropertyChanged 通知对象属性变化。绑定控件更新应在 UI 线程进行。
- 学习代码为模拟设备管理，不连接真实 PLC；字符串点位格式检查不等于验证硬件映射有效。

## 学习状态

待学习 / 待验证。用户已完成设备管理主界面设计器布局并提供截图；尚未看到完整 Form2 / Form1 代码、编译结果或运行时数据绑定验证。
