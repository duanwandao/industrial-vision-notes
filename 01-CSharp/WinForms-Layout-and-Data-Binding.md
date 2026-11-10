# WinForms 布局、对话框与数据绑定

> 学习日期：2026-11-09  
> 状态：待学习 / 待验证。当前内容来自文字讲解和设计器练习指导；尚未确认所有代码已在用户项目中独立编译运行。  
> 适用：.NET 8 WinForms；具体行为以目标 .NET Windows Forms 版本为准。

## 1. WinForms 容器布局

### TableLayoutPanel

以行列网格组织子控件，适合主窗体区域划分和表单字段对齐。

常用属性：
- `ColumnCount` / `RowCount`：列数、行数。
- `ColumnStyles` / `RowStyles`：列/行尺寸策略。
- `Dock`：相对父容器停靠；`DockStyle.Fill` 填满父容器可用区域。
- `Padding`：容器内部留白。
- `CellBorderStyle`：单元格边框。
- `GrowStyle`：网格空间不足时的扩展行为。

三种尺寸策略：
- `Absolute`：固定像素尺寸，适合固定宽度侧栏、标题栏/状态栏。
- `Percent`：按可分配剩余空间比例分配；不是无条件按整个窗体尺寸计算，因为 Absolute、AutoSize 等也会占空间。
- `AutoSize`：根据内容所需尺寸调整行/列。

常见结构：根 TableLayoutPanel 划分 Header / Content / Footer；Content 内再嵌套一个 TableLayoutPanel 划分左侧参数区、中央图像区和右侧结果区。通过 `SetColumnSpan(control, count)`、`SetRowSpan(control, count)` 让控件跨列/跨行。

### FlowLayoutPanel

按流向依次排列子控件，不需要预先定义行列，适合工具栏、按钮组、动态标签。

- `FlowDirection`：`LeftToRight`、`TopDown`、`RightToLeft`、`BottomUp`。
- `WrapContents`：空间不足时是否换行。
- `AutoScroll`：内容超出可视范围时提供滚动。
- `Padding`：面板内部留白。
- 子控件的 `Margin`：控件外侧布局留白。
- `AutoSize`：根据内容调整面板自身尺寸；固定高度工具栏通常不需要开启。
- `SetFlowBreak(control, true)`：在指定控件后设置换行点；LeftToRight 下后续控件进入下一行，TopDown 下进入下一列。

典型横向工具栏：
```csharp
var toolbar = new FlowLayoutPanel
{
    Dock = DockStyle.Top,
    Height = 60,
    FlowDirection = FlowDirection.LeftToRight,
    WrapContents = false,
    AutoScroll = true,
    Padding = new Padding(8)
};
```

### Panel 与组合原则

- `Panel`：普通容器，适合承载一组控件或局部区域。
- `TableLayoutPanel`：负责区域网格划分。
- `FlowLayoutPanel`：负责重复控件的流式排列。
- 复杂窗体采用嵌套容器，不要把所有控件直接绝对定位在 Form 上。
- `Dock` 决定停靠/填充；`Anchor` 维持控件与父容器边缘的相对关系。二者都不是数据绑定机制。

## 2. DataGridView 基础与数据绑定

DataGridView 用于展示/编辑二维数据。核心概念：
- Column：列；Row：行；Cell：单元格。
- `Columns.Add(name, headerText)` 添加列。
- `Rows.Add(values...)` 在非绑定模式下添加行。
- `DataSource` 指定绑定数据源。
- `DataPropertyName` 指定列映射到模型的哪个公开属性。
- `HeaderText` 是用户看到的列标题，不等于模型属性名。
- `nameof(Model.Property)` 在编译期引用属性名称，减少硬编码字符串。

常见设置：
```csharp
dgvResults.ReadOnly = true;
dgvResults.AllowUserToAddRows = false;
dgvResults.AllowUserToDeleteRows = false;
dgvResults.RowHeadersVisible = false;
dgvResults.SelectionMode = DataGridViewSelectionMode.FullRowSelect;
dgvResults.MultiSelect = false;
dgvResults.AutoSizeColumnsMode = DataGridViewAutoSizeColumnsMode.Fill;
```

两种使用方式：
1. 非绑定模式（Unbound）：手动创建列并调用 `Rows.Add`，适合简单演示或完全由控件管理的小表格。
2. 数据绑定模式：将对象集合、BindingSource、DataTable 等设为 `DataSource`，数据模型与界面职责分离。真实业务记录通常优先采用数据绑定。

对象绑定示例：
```csharp
public class InspectionRecord
{
    public int Index { get; set; }
    public string ProductCode { get; set; } = "";
    public DateTime InspectionTime { get; set; }
    public string Result { get; set; } = "";
    public double DurationMs { get; set; }
}

dgvResults.AutoGenerateColumns = false;
dgvResults.Columns.Add(new DataGridViewTextBoxColumn
{
    Name = "colProductCode",
    HeaderText = "产品编号",
    DataPropertyName = nameof(InspectionRecord.ProductCode)
});
dgvResults.DataSource = records;
```

`CellFormatting` 可根据单元格值设置显示样式，例如 OK/NG 文字颜色。格式化事件用于显示，不应把它当作业务数据修改入口。

## 3. 模态对话框：DialogResult、ShowDialog、Owner

适合“新增设备参数”“编辑设备参数”等需要用户先完成/取消的操作。

### DialogResult

`DialogResult` 是枚举，常见值包括 `None`、`OK`、`Cancel`、`Yes`、`No` 等。

Form2 内：
```csharp
private void btnOK_Click(object? sender, EventArgs e)
{
    // 先校验并准备返回数据
    CreatedLight = new Light { Name = name, Address = address };
    DialogResult = DialogResult.OK;
}
```

在模态对话框中，将窗体的 `DialogResult` 属性设置为非 `None` 的值会结束对话框；调用方的 `ShowDialog` 返回该结果。通常无需在设置 DialogResult 后再显式调用 `Close()`。取消按钮可设置 `DialogResult.Cancel`；也可在设计器中设置窗体的 `AcceptButton` / `CancelButton`，让 Enter / Esc 触发相应按钮。

注意左右同名：`this.DialogResult` 是当前窗体属性，`DialogResult.OK` 是枚举成员。

### ShowDialog 与 Owner

- `Show()` 显示非模态窗口，调用后继续执行，主窗口仍可交互。
- `ShowDialog()` 显示模态窗口，调用线程等待用户关闭对话框后才继续执行。
- `ShowDialog(this)` 把当前窗体作为 Owner（拥有者窗口），用于建立窗口所有权关系；通常使对话框保持在拥有者前面，并支持 `CenterParent` 等定位行为。这里的 `this` 是调用代码所在窗体实例，不是传递业务数据。

### using 与 Dispose

```csharp
using var dialog = new Form2();
if (dialog.ShowDialog(this) != DialogResult.OK)
    return;
```

`var` 根据 `new Form2()` 推断为 Form2；`using var` 是 using 声明，编译器生成作用域结束时调用 `Dispose()` 的清理逻辑（包含异常或提前 return 路径）。它不是立即释放，也不是自动创建后台线程。

近似理解：
```csharp
Form2 dialog = new Form2();
try
{
    dialog.ShowDialog(this);
}
finally
{
    dialog.Dispose();
}
```

模态窗体结束后，调用方应释放临时窗体资源。Dispose Form2 不会自动 Dispose 从它取出并存入主窗体集合的业务对象 Light。

## 4. 窗体之间传递数据：返回对象，而非直接操作对方控件

推荐职责：
- Form1：拥有设备集合、执行重复地址检查、管理列表和设备操作。
- Form2：收集并验证单个设备的参数，创建/返回数据对象。
- Form2 不应直接访问 Form1 的 ListBox 或私有集合。

示例：
```csharp
// Form2
public Light? CreatedLight { get; private set; }

// 验证成功后
CreatedLight = new Light
{
    Name = name,
    Address = address,
    IsOn = false
};
DialogResult = DialogResult.OK;

// Form1
using var dialog = new Form2();
if (dialog.ShowDialog(this) != DialogResult.OK)
    return;

if (dialog.CreatedLight is not Light light)
    return;

if (_lights.Any(x => string.Equals(
        x.Address, light.Address,
        StringComparison.OrdinalIgnoreCase)))
{
    MessageBox.Show("该 PLC 点位已经被使用！");
    return;
}

_lights.Add(light);
```

`dialog.CreatedLight is not Light light` 是类型模式匹配：
- 检查属性值是否为非 null 的 Light 对象。
- 匹配成功时声明局部变量 `light`，它引用 Form2 已创建的同一个对象。
- 这里没有执行 `new Light()`，没有复制对象。
- 若匹配失败，条件成立并可提前 return；通过检查后，变量可安全使用。

`Any(predicate)` 判断集合是否至少有一个元素满足条件；地址比较使用 `OrdinalIgnoreCase` 可忽略大小写。格式校验（如正则）不等于验证地址在具体 PLC 硬件配置中真实存在，硬件映射仍须核对项目配置/手册。

## 5. ListBox 的 BeginUpdate / EndUpdate

在非数据绑定模式下批量重建 Items 时，可以减少中间重绘和闪烁：
```csharp
lstLights.BeginUpdate();
try
{
    lstLights.Items.Clear();
    foreach (var light in lights)
        lstLights.Items.Add(light);
}
finally
{
    lstLights.EndUpdate();
}
```

- `BeginUpdate()` 暂停该控件的重绘更新，不会暂停 UI 线程，不创建线程，也不是线程锁。
- `EndUpdate()` 恢复绘制；必须保证成对调用。
- 使用 `try/finally`，确保中途异常时也执行 EndUpdate。
- 它是具体控件提供的方法，不要假设所有 WinForms 控件都有 BeginUpdate/EndUpdate。
- 若数据源已绑定且通过 BindingList 通知增删，通常不需要清空并重建 Items；应直接修改数据源。

## 6. BindingList<T>：集合变化通知

`List<T>` 是通用集合；新增/删除元素不会主动向 WinForms 绑定控件发出集合变化通知。可以使用：
```csharp
using System.ComponentModel;

private readonly BindingList<Light> _lights = new();
lstLights.DataSource = _lights;
```

绑定后通过数据源操作：
```csharp
_lights.Add(light);
_lights.Remove(light);
_lights.Clear();
```

不要再对同一个已绑定 ListBox 调用 `Items.Add`、`Items.Clear` 来管理绑定数据；数据应由 DataSource 管理。

ListBox 可配置：
```csharp
lstLights.DisplayMember = nameof(Light.DisplayText);
lstLights.DataSource = _lights;
```

`DisplayMember` 指定对象用哪个属性显示。另一种简单方式是重写 `ToString()`，但显式 DisplayMember 通常更清楚，也方便 UI 显示文本与对象模型分离。

BindingList<T> 实现了绑定集合所需的通知机制，能通知绑定控件新增、删除、重置等集合变化。它本身不自动把任意对象的所有属性改动都变成通知；要让条目属性变化也更新界面，模型通常还需实现 INotifyPropertyChanged。

## 7. INotifyPropertyChanged：对象属性变化通知

`INotifyPropertyChanged` 位于 `System.ComponentModel`，定义 `PropertyChanged` 事件。对象属性 setter 在值真正变化后触发事件，通知绑定层哪个属性变了。

```csharp
using System.ComponentModel;
using System.Runtime.CompilerServices;

public class Light : INotifyPropertyChanged
{
    private string _name = "";
    private string _address = "";
    private bool _isOn;

    public event PropertyChangedEventHandler? PropertyChanged;

    public string Name
    {
        get => _name;
        set
        {
            if (_name == value) return;
            _name = value;
            OnPropertyChanged();
            OnPropertyChanged(nameof(DisplayText));
        }
    }

    public string Address
    {
        get => _address;
        set
        {
            if (_address == value) return;
            _address = value;
            OnPropertyChanged();
            OnPropertyChanged(nameof(DisplayText));
        }
    }

    public bool IsOn
    {
        get => _isOn;
        set
        {
            if (_isOn == value) return;
            _isOn = value;
            OnPropertyChanged();
            OnPropertyChanged(nameof(DisplayText));
        }
    }

    public string DisplayText =>
        $"{Name}（{Address}）- {(IsOn ? "亮" : "灭")}";

    private void OnPropertyChanged(
        [CallerMemberName] string? propertyName = null)
    {
        PropertyChanged?.Invoke(
            this, new PropertyChangedEventArgs(propertyName));
    }
}
```

关键点：
- 私有字段保存状态；属性 setter 是统一修改入口。
- 先比较新旧值，避免无意义通知。
- `[CallerMemberName]` 让编译器自动传入调用成员名称；在 Name setter 内调用时，参数默认是 `"Name"`。
- `PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName))` 通知订阅者。
- `DisplayText` 是计算属性，不直接存储值；它依赖 Name、Address、IsOn。因此这些属性变化时，除了通知自身属性，还要通知 DisplayText。
- 当 BindingList<T> 中的元素实现 INotifyPropertyChanged 时，BindingList 会订阅条目属性事件并将其转为列表项变化通知，绑定控件据此刷新对应显示。

线程边界：绑定通知和 WinForms 控件更新应在 UI 线程执行。若工业相机 SDK 在后台线程回调，不要直接从回调线程修改绑定集合或控件；应先通过 Invoke/BeginInvoke 或适当的 UI 调度机制切回 UI 线程。

## 8. 推荐练习：Light 管理对话框

模型：
```csharp
public class Light : INotifyPropertyChanged
{
    // Name、Address、IsOn、DisplayText 和通知逻辑见上方完整示例
}
```

Form1：
- `BindingList<Light> _lights` 作为唯一集合数据源。
- `lstLights.DisplayMember = nameof(Light.DisplayText)`。
- `lstLights.DataSource = _lights`。
- 新增：Form2 返回 CreatedLight，Form1 校验地址唯一后 `_lights.Add(light)`。
- 删除：从 SelectedItem 模式匹配出 Light，再 `_lights.Remove(light)`。
- 清空：`_lights.Clear()`。
- 点亮/熄灭：修改选中对象的 `IsOn`，验证列表显示是否自动变化。
- 若有独立的选中设备 Label，仍需单独更新它；数据绑定不会自动更新未绑定的控件。

验证清单：
1. 点击取消，主集合不增加对象。
2. 点击确定，Form1 收到 Form2 创建的同一个 Light 对象。
3. 重复 PLC 地址（大小写不同也算重复）被拒绝。
4. BindingList 新增/删除/清空时，ListBox 与数量显示同步。
5. 修改 IsOn / Name 后，DisplayText 更新。
6. 模拟后台回调时，先切回 UI 线程再更新集合。
7. 不把软件按钮状态当作 PLC 输出真实反馈；实际灯状态应读取设备反馈/状态，不应仅凭本地 IsOn 推断硬件已动作。

## 9. 易混淆点速查

| 概念 | 解决的问题 |
|---|---|
| TableLayoutPanel | 按行列划分区域 |
| FlowLayoutPanel | 按流向排列控件 |
| Panel | 普通控件容器 |
| Dock / Anchor | 控件相对父容器的布局关系 |
| DataGridView.DataSource | 表格数据来源 |
| DialogResult | 模态对话框返回结果 |
| ShowDialog(this) | 显示模态窗口并指定 Owner |
| using var | 作用域结束时自动 Dispose |
| BeginUpdate / EndUpdate | 批量更新时控制控件重绘 |
| BindingList<T> | 集合增删变化通知 |
| INotifyPropertyChanged | 对象属性变化通知 |
| DisplayMember | 指定列表控件显示对象的哪个属性 |


## 10. BindingSource：WinForms 数据绑定中间层

`BindingSource` 位于底层数据源与 WinForms 控件之间，提供统一绑定入口、当前项、当前位置和导航；它不是另一份业务集合，也不替代 `BindingList<T>`。

用户的 Designer 截图已经有 `lightBindingSource = new BindingSource(components);`，所以直接使用这个设计器组件，不要再 new 一个重复实例。控件/组件在 `InitializeComponent()` 中创建，配置放在它之后：

```csharp
private readonly BindingList<Light> _lights = new();

public Form1()
{
    InitializeComponent();

    lightBindingSource.DataSource = _lights;
    lstLights.DisplayMember = nameof(Light.DisplayText);
    lstLights.DataSource = lightBindingSource;
}
```

常用成员：`DataSource`（底层源）、`Current`（当前对象）、`Position`（当前索引，通常从 0 开始）、`Count`、`IndexOf(item)`、`MoveFirst/MovePrevious/MoveNext/MoveLast`、`RemoveCurrent()`、`CurrentChanged`。

`Current` 可能为 null，使用前做模式匹配：

```csharp
if (lightBindingSource.Current is not Light selected)
{
    return;
}
```

添加后选中新对象：

```csharp
_lights.Add(light);
int index = lightBindingSource.IndexOf(light);
if (index >= 0)
    lightBindingSource.Position = index;
```

删除仍可操作唯一数据源：

```csharp
if (lightBindingSource.Current is Light selected)
    _lights.Remove(selected);
```

### BindingSource.Filter / Sort 的能力边界

BindingSource 提供 `Filter` / `Sort`，但取决于底层数据源是否真正支持：

- Filter 要求底层实现 `IBindingListView` 并支持过滤（`SupportsFiltering`）。
- Sort 要求底层 `IBindingList` / `IBindingListView` 实际支持排序（检查 `SupportsSorting`）。
- 普通 `BindingList<T>` 虽实现 `IBindingList`，但默认 `SupportsSortingCore == false`，`ApplySortCore` 默认抛 `NotSupportedException`；它也没有内建 `IBindingListView` 过滤实现。因此不能假设 BindingSource 的 Filter/Sort 对普通 BindingList 可用。
- `DataView` 支持过滤和排序；以 DataTable/DataView 为底层时可以使用对应视图能力。

```csharp
bool canFilter = lightBindingSource.SupportsFiltering;
bool canSort = lightBindingSource.SupportsSorting;
```

对当前 `BindingList<Light>` 项目，先用 LINQ 做查询。`Where/OrderBy/ToList` 结果是新 List 快照，不会自动跟源 BindingList 后续增删同步；若要持续绑定过滤结果，应维护独立的可观察显示集合并在源数据变化时重新应用筛选，或实现明确支持排序/过滤的绑定列表。

## 11. BindingList<T> 的 ListChanged 与属性事件源码

.NET 8 Runtime 的 `BindingList<T>` 使用私有委托字段保存订阅者，并通过事件访问器管理：

```csharp
[NonSerialized]
private ListChangedEventHandler? _onListChanged;

public event ListChangedEventHandler ListChanged
{
    add => _onListChanged += value;
    remove => _onListChanged -= value;
}

protected virtual void OnListChanged(ListChangedEventArgs e)
{
    _onListChanged?.Invoke(this, e);
}
```

外部执行 `list.ListChanged += Handler` 会进入 `add`；执行 `-=` 会进入 `remove`。`value` 是本次传入的事件处理委托。只有 BindingList 内部的触发路径调用 `OnListChanged`。

BindingList 重写集合操作发出通知：`InsertItem` 加入后发出 `ItemAdded`；`RemoveItem` 删除前解除条目属性事件订阅，再发出 `ItemDeleted`；`ClearItems` 清空前解除所有订阅，再发出 `Reset`；`SetItem` 替换时解除旧对象、订阅新对象，并发出 `ItemChanged`。`RaiseListChangedEvents` 可暂时抑制通知，但不是线程锁。

### BindingList 在哪里监听 INotifyPropertyChanged？

.NET 8 源码中的核心方法：

```csharp
private void HookPropertyChanged(T item)
{
    if (item is INotifyPropertyChanged inpc)
    {
        _propertyChangedEventHandler ??=
            new PropertyChangedEventHandler(Child_PropertyChanged);
        inpc.PropertyChanged += _propertyChangedEventHandler;
    }
}

private void UnhookPropertyChanged(T item)
{
    if (item is INotifyPropertyChanged inpc
        && _propertyChangedEventHandler != null)
    {
        inpc.PropertyChanged -= _propertyChangedEventHandler;
    }
}
```

BindingList 不是轮询 Light 属性，而是在对象加入时订阅 `PropertyChanged`；删除、替换、清空时解除订阅。`Child_PropertyChanged` 定位条目并向绑定层发出 `ItemChanged`（事件参数不完整时可能发出 Reset）。所以：BindingList 通知集合变化；INotifyPropertyChanged 通知对象属性变化；BindingList 将条目属性事件转换为 ListChanged。

源码：[.NET 8 BindingList.cs](https://github.com/dotnet/runtime/blob/v8.0.0/src/libraries/System.ComponentModel.TypeConverter/src/System/ComponentModel/BindingList.cs)。

## 12. C# is、not、or 与 ||

`is` 可做类型检查，也可用类型模式声明变量：

```csharp
if (value is Light light)
{
    // light 指向 value 原来引用的对象；没有 new，也没有复制。
}
if (value is not Light light)
{
    return;
}
```

`not` / `or` 是模式组合符：

```csharp
if (status is InspectionStatus.NG or InspectionStatus.Error)
{
}
```

`or` 只能在模式匹配语境中组合模式；`||` 是普通 bool 短路逻辑或：

```csharp
if (status == InspectionStatus.NG ||
    status == InspectionStatus.Error)
{
}
```

Java 的 `instanceof` 与 C# 的 `is` 有相似用途；Java/JavaScript 的 `||` 对应普通逻辑或，但没有完全相同的 C# `not` / `or` 模式组合语法。

C# 不是把 C++、Java、JavaScript 简单拼接，也不是把设计模式变成关键字。它采用 C 家族熟悉的语法，同时提供静态类型、面向对象、泛型、委托/事件、Lambda、LINQ、异步和模式匹配等能力。设计模式仍是软件设计思想，语言特性让部分模式更容易表达。

## 13. event 与普通委托：发布者保留触发权

`delegate` 是可调用对象类型；多播委托可保存多个处理器。`event` 在委托调用列表外增加访问限制：

```csharp
public class Camera
{
    public event EventHandler? ImageCaptured;

    public void CaptureCompleted()
    {
        ImageCaptured?.Invoke(this, EventArgs.Empty);
    }
}
```

外部可 `+=` 订阅、`-=` 退订，但不能直接 `Invoke` 事件，也不能把事件赋值为 null 清空所有订阅者。只有声明事件的类型内部负责触发。

概念上，事件的自定义访问器相当于管理私有委托字段：

```csharp
private EventHandler? _changed;

public event EventHandler Changed
{
    add => _changed += value;
    remove => _changed -= value;
}

protected virtual void OnChanged()
{
    _changed?.Invoke(this, EventArgs.Empty);
}
```

`+=` / `-=` 是注册和取消注册，不是立即执行处理器。普通公开委托字段可能允许外部直接调用、覆盖或清空调用列表。普通事件处理器通常在触发事件的线程同步执行；event 不创建线程，也不自动切换到 WinForms UI 线程。

## 14. string.Empty、String.Empty、null 与 null!

`string` 是 C# 关键字别名，对应 `System.String`；`string.Empty` 与 `String.Empty` 都可用。

- `""` 与 `string.Empty` 都是长度为 0 的非 null 字符串，语义等价，选择一种风格即可。
- `null` 表示没有字符串引用，与空字符串不同。
- `string?` 是可空引用类型注解，主要影响编译器静态分析，不会创建新的运行时字符串类型。
- `null!` 只抑制可空性警告，运行时值仍然是 null；它不会创建空字符串。

```csharp
private string _name = string.Empty;
private string? _optionalDescription = null;
private Button btnAdd = null!; // Designer 在 InitializeComponent 中稍后创建
```

业务对象若能在构造时提供必需名称，优先通过构造参数建立有效对象，不要滥用 `null!`。`string.Empty` 也不代表输入已通过业务验证，仍需使用 `string.IsNullOrWhiteSpace` 等规则。

## 15. ToString、DisplayMember 与 DisplayText

`object.ToString()` 是虚方法。类不重写时通常得到类型名称；重写后可提供有意义的文本：

```csharp
public override string ToString() => Name + " - " + Address;
```

ListBox 未设置 `DisplayMember` 时通常使用对象 `ToString()`；设置 `DisplayMember` 后读取指定公开属性，不会因为重写 ToString 就自动拼接多个字段。

推荐使用计算属性：

```csharp
public string DisplayText => Name + " - " + Address;
public override string ToString() => DisplayText;
```

绑定：

```csharp
lstLights.DisplayMember = nameof(Light.DisplayText);
lstLights.DataSource = lightBindingSource;
```

`DisplayMember` 一次指定一个属性；`DisplayText` 可以组合 Name 和 Address。DisplayText 依赖这两个属性，因此 Name 或 Address setter 修改成功后，还要触发 `OnPropertyChanged(nameof(DisplayText))`。ToString 更适合调试、日志和通用文本表示；DisplayText/DisplayMember 更适合 UI 展示契约。

## 16. 字段初始化器与构造函数初始化

不依赖控件的集合可以在字段声明时初始化：

```csharp
private readonly BindingList<Light> _lights = new();
```

字段初始化器在构造函数主体执行前完成。WinForms 设计器控件和组件通常在 `InitializeComponent()` 中创建，因此不要在字段初始化器中访问尚未创建的 `lstLights` 或 Designer 组件。应先执行 `InitializeComponent()`，再设置 DataSource / DisplayMember。如果 BindingSource 已由 Designer 通过 `new BindingSource(components)` 创建，就直接配置这个实例。

## 17. 对话框验证与编辑交互：重复 PLC 地址时保留输入

用户当前 `btnAdd_Click` 截图显示：先 `dialog.ShowDialog(this)`，对话框返回 OK 并关闭后，Form1 才用 LINQ `Any` 检查地址是否重复。这样发现重复时参数窗口已经关闭，用户必须重新打开并重填。

若希望验证失败后保留输入，校验必须在 Form2 设置 `DialogResult.OK` 之前：

1. Form2 读取并 Trim 名称、地址，检查必填、格式和重复。
2. 失败时显示 ErrorProvider/就地错误，聚焦地址框并 SelectAll，然后 return；不要设置 DialogResult.OK。
3. 成功后创建待提交 Light，最后设置窗体 `DialogResult = DialogResult.OK`。
4. 确认按钮的 Button.DialogResult 设为 `None`，由点击处理器在验证成功后设置窗体结果；Form 的 `AcceptButton` 可指向它。取消按钮可设 Cancel 并作为 `CancelButton`。

重复判断示例：

```csharp
bool duplicate = existingLights.Any(x =>
    !ReferenceEquals(x, editingLight) &&
    string.Equals(x.Address?.Trim(), address.Trim(),
        StringComparison.OrdinalIgnoreCase));
```

`Any` 找到至少一个满足条件的元素就返回 true，并可提前结束。编辑时用 `ReferenceEquals` 排除正在编辑的对象，否则自己的原地址也会被判重复。若 Address 保证非 null，可移除 `?.`。

编辑时先在对话框输入框中修改草稿，确认成功后返回新 Light；Form1 再将允许修改的字段复制到原对象，以保留对象身份和当前选择，并利用 INotifyPropertyChanged 更新绑定显示。取消时不修改原对象。不要在普通确认后额外询问是否关闭；只有用户修改过内容后又取消/点 X，才考虑询问是否放弃未保存修改。PLC 地址格式和软件集合唯一性检查不证明点位在目标 PLC 工程中真实存在。

当前项目截图是 ListBox，不需要为了学习 BindingSource 先添加 DataGridView。

## 参考资料

- [Microsoft Learn：DataGridView.DataSource](https://learn.microsoft.com/en-us/dotnet/api/system.windows.forms.datagridview.datasource)
- [Microsoft Learn：绑定对象到 DataGridView](https://learn.microsoft.com/en-us/dotnet/desktop/winforms/controls/how-to-bind-objects-to-windows-forms-datagridview-controls)
- [Microsoft Learn：INotifyPropertyChanged](https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.inotifypropertychanged)
