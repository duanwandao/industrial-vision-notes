# 2026-11-10 BindingSource、事件源码与灯光管理交互

## 用户问题与当前项目证据

- 继续询问 BindingSource 的 Filter / Sort 是否能直接筛选和排序。
- 询问 C# 是否像 C++、Java、JavaScript 的结合体，以及语言特性与设计模式的关系。
- 当前 btnAdd_Click 先 ShowDialog，窗口关闭后才用 LINQ Any 检查 PLC Address 是否重复；希望失败时保留输入并留在对话框。
- 希望 ListBox 显示 Light 名称 + Address，询问 ToString 与 DisplayMember 的关系。
- 询问字段初始化器与构造函数配置的区别。
- 继续学习 is、or、||、BindingList.ListChanged 源码、BindingList 如何订阅 Light.PropertyChanged、BindingSource。
- 用户截图确认主窗体当前使用 ListBox，没有添加 DataGridView。
- Form1.Designer.cs 已由设计器创建 lightBindingSource = new BindingSource(components)，但尚无证据表明它已在 Form1 构造函数接入数据源。

## 整理要点

1. BindingSource 是绑定中间层，提供 Current、Position、Count、IndexOf、导航、RemoveCurrent；不替代 BindingList。
2. Filter 依赖 IBindingListView 过滤支持；Sort 依赖底层 IBindingList / IBindingListView 的排序支持。普通 BindingList<T> 默认不支持 Sort，也没有 Filter 实现；DataView 支持过滤和排序。
3. .NET 8 BindingList<T> 的 ListChanged 事件使用私有委托字段和 add/remove 访问器；OnListChanged 触发订阅者。InsertItem/RemoveItem/ClearItems/SetItem 负责集合通知。
4. HookPropertyChanged 在条目加入时订阅 INotifyPropertyChanged.PropertyChanged；移除/替换/清空时 Unhook；Child_PropertyChanged 将属性变化转换为 ItemChanged/Reset。
5. is 类型模式可同时声明变量；not/or 是模式组合符；|| 是普通 bool 短路逻辑或。
6. string 是 System.String 别名；string.Empty 与 "" 是非 null 空字符串；null 表示无引用；null! 只抑制编译器警告，运行时仍为 null。
7. ListBox DisplayMember 读取指定公开属性；显示 Name + Address 应使用 DisplayText 计算属性，并在依赖属性变化时通知 DisplayText。ToString 可用于调试/默认显示。
8. _lights 可在字段声明时 new；Designer 控件和 lightBindingSource 在 InitializeComponent 中创建，DataSource 等配置放在其后。
9. 重复地址校验应在 Form2 关闭前完成；确认按钮 DialogResult=None，失败时显示错误并保留输入，成功后才设置窗体 DialogResult.OK。编辑时排除当前对象自身，先编辑草稿，确认后提交。
10. 当前改造尚未确认在项目中独立编译运行；未连接真实 PLC。

## 状态

待学习 / 待验证。
