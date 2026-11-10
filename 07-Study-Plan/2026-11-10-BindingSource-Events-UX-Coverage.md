# 2026-11-10 BindingSource、事件源码与灯光管理交互覆盖报告

> 状态：待学习 / 待验证。依据文字讨论与用户截图整理；未声称代码已独立编译运行或连接真实设备。

## 覆盖范围

- BindingSource 的 DataSource、Current、Position、Count、IndexOf、导航、RemoveCurrent。
- BindingSource Filter / Sort 的底层接口条件；普通 BindingList<T> 默认不支持过滤/排序；DataView 支持相应视图操作。
- .NET 8 BindingList<T> 的 ListChanged add/remove、OnListChanged、FireListChanged、集合操作通知。
- HookPropertyChanged / UnhookPropertyChanged：BindingList 订阅/解除订阅条目 INotifyPropertyChanged；Child_PropertyChanged 转发 ItemChanged/Reset。
- C# is/not/or 模式与 ||；event 与普通委托；string.Empty/null/null!。
- ToString、DisplayMember、DisplayText 计算属性；字段初始化器与构造函数时机。
- 对话框内地址重复验证、编辑排除自身、草稿提交与取消语义。

## 用户项目证据与边界

- 当前主窗体使用 ListBox，没有证据表明已经添加 DataGridView。
- Designer 截图显示已创建 lightBindingSource = new BindingSource(components)，但尚未确认构造函数已完成 DataSource 绑定。
- btnAdd_Click 截图显示：ShowDialog 返回 OK 后，Form1 才执行 _lights.Any(...) 地址重复检查；重复提示出现时参数窗口已经关闭。
- 用户界面已有添加、修改、删除、清空和灯光控制区域；本轮建议尚未在项目运行验证。
- 没有真实 PLC 工程映射验证；字符串地址检查不证明点位在硬件上存在或接线正确。

## 关键结论

1. BindingSource 是中间层，不是新的业务集合。
2. 普通 BindingList<T> 的 SupportsSortingCore 默认 false，ApplySortCore 默认不支持；没有内建 IBindingListView 过滤支持。
3. BindingList 通过事件订阅而非轮询检测条目属性变化。
4. DisplayText 组合 Name 和 Address；修改依赖属性时也要通知 DisplayText。
5. 验证失败不能设置 DialogResult.OK；成功后才关闭模态对话框。
6. 编辑草稿确认后再提交，取消不应污染原对象。

## 下一步实践

1. 在 InitializeComponent() 后设置 lightBindingSource.DataSource = _lights。
2. 设置 lstLights.DisplayMember = nameof(Light.DisplayText)，lstLights.DataSource = lightBindingSource。
3. 通过 Current 获取选中 Light；新增后用 IndexOf + Position 选中。
4. 用 ListChanged 更新数量标签；用 CurrentChanged 更新右侧详情。
5. 把重复地址验证移到 Form2 确认前；失败保留输入，编辑时排除自身。
6. 测试 Add / Remove / Clear、Name / Address / IsOn 变化、取消不改原对象；所有绑定集合更新都在 UI 线程执行。

## 参考资料

- [.NET 8 BindingList<T> 源码](https://github.com/dotnet/runtime/blob/v8.0.0/src/libraries/System.ComponentModel.TypeConverter/src/System/ComponentModel/BindingList.cs)
- [BindingSource](https://learn.microsoft.com/dotnet/api/system.windows.forms.bindingsource)
- [BindingSource.Filter](https://learn.microsoft.com/dotnet/api/system.windows.forms.bindingsource.filter)
- [BindingSource.Sort](https://learn.microsoft.com/dotnet/api/system.windows.forms.bindingsource.sort)
- [C# 模式匹配](https://learn.microsoft.com/dotnet/csharp/language-reference/operators/patterns)
- [C# 事件](https://learn.microsoft.com/dotnet/csharp/events-overview)
