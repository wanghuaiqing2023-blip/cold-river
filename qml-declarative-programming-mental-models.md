# QML 声明式 UI 的心智模型笔记

> 这份笔记整理自一次围绕 KDE Plasma / QML / FolderItemDelegate.qml 的调试讨论，目标是帮助有命令式编程背景的程序员建立 QML 的正确心智模型。

## 1. 总览：QML 不是“顺序执行脚本”

初学 QML 时，最容易把它当成 JavaScript 或 C++ 那样的顺序执行代码：

```qml
Rectangle {
    width: 100
    height: 50
}
```

更合适的理解是：

```text
QML 文件声明了一棵对象树；
属性之间形成依赖图；
状态变化会驱动绑定自动重新计算；
事件处理器里才写命令式逻辑。
```

一句话总结：

```text
QML = 对象树 + 属性绑定 + 响应式更新 + 模板实例化 + 少量事件回调
```

---

## 2. 四层来源：哪些是语言，哪些是框架，哪些是业务代码

阅读 QML 文件时，可以把名字分成四层。

### 2.1 QML 语言机制

这些是 QML 语言本身的机制：

```qml
id
property
signal
function
属性名: 表达式
onXXXChanged
Component.onCompleted
```

例如：

```qml
Rectangle {
    id: box
    property bool fixedWidth: false

    width: fixedWidth ? 300 : parent.width / 2

    onWidthChanged: {
        console.log(width)
    }
}
```

其中：

```text
id                 QML 对象局部名字
property           声明属性
width: ...         属性绑定
onWidthChanged     属性变化信号处理器
console.log        QML/JS 调试输出
```

### 2.2 Qt Quick 提供的类型

这些通常来自：

```qml
import QtQuick 2.x
```

常见类型包括：

```text
Item
Rectangle
Text
TextInput
MouseArea
ListView
GridView
Loader
Component
```

### 2.3 KDE Plasma 提供的类型

这些通常来自：

```qml
import org.kde.plasma.core as PlasmaCore
import org.kde.plasma.components as PlasmaComponents
```

例如：

```qml
PlasmaCore.ToolTipArea
PlasmaComponents.Label
PlasmaCore.Types.LeftEdge
plasmoid
```

其中 `plasmoid` 通常是 Plasma 运行环境注入的上下文对象。

### 2.4 当前 QML 文件或 Plasma 桌面业务代码定义的名字

例如在 `FolderItemDelegate.qml` 中：

```qml
id: main
id: label
id: icon
id: toolTip
id: minimalToolTip
```

这些 `id` 是文件作者起的局部对象名，不是语言关键字。

---

## 3. `id` 的心智模型：不是字符串，不是普通变量

```qml
Rectangle {
    id: box
    width: 100
    height: 50
}
```

不要理解成：

```js
id = "box"
```

而要理解成：

```text
创建一个 Rectangle 对象；
在当前 QML 作用域内，把这个对象命名为 box。
```

所以：

```qml
target: box
```

意思是：

```text
target 属性引用 box 这个对象。
```

注意：

```qml
id: box          // 对
id: "box"        // 错
id: red-box      // 错
id: red_box      // 对
```

`id` 要像变量名一样，不能加引号，不能有空格或横线，并且同一作用域内不能重复。

---

## 4. `属性: 表达式` 是绑定，不是一次性赋值

这是理解 QML 的核心。

```qml
Rectangle {
    id: box
    width: parent.width / 2
}
```

它不是启动时计算一次，而是：

```text
box.width 长期由 parent.width / 2 决定。
parent.width 变化时，box.width 自动重新计算。
```

可以类比 Excel：

```text
A3 = A1 + A2
```

当 A1 或 A2 变化时，A3 自动变化。

QML 中也是类似：

```qml
width: parent.width / 2
```

表示：

```text
width 是一个依赖 parent.width 的公式。
```

---

## 5. 条件表达式仍然是绑定

例如：

```qml
Rectangle {
    id: box

    property bool fixedWidth: false

    width: fixedWidth ? 300 : parent.width / 2
}
```

这仍然是绑定关系。

它不是：

```text
width 只绑定到 parent.width / 2
```

而是：

```text
width 绑定到整个表达式：
fixedWidth ? 300 : parent.width / 2
```

运行时可以这样理解：

```js
function computeWidth() {
    return fixedWidth ? 300 : parent.width / 2
}
```

当 `fixedWidth` 或 `parent.width` 变化时，QML 引擎会重新计算 `width`。

如果：

```qml
box.fixedWidth = true
```

那么 `width` 变成 300。此时 `parent.width` 再变化，`box.width` 不跟着变，不是绑定断了，而是因为当前公式分支本来就不使用 `parent.width`。

如果再执行：

```qml
box.fixedWidth = false
```

`width` 又重新回到 `parent.width / 2` 的分支。

---

## 6. 命令式赋值会打断绑定

假设：

```qml
Rectangle {
    id: box
    width: parent.width / 2
}
```

这里 `width` 绑定到 `parent.width / 2`。

如果事件里写：

```qml
box.width = 300
```

这会把原来的绑定覆盖掉。

类比 Excel：

```text
单元格里原来是公式：=A1/2
你手动输入 300
公式被覆盖了
```

所以 QML 中要谨慎直接修改“结果属性”。

---

## 7. 正确思路：修改状态，不直接修改结果

不推荐：

```qml
Label {
    id: label
    text: model.display
}

onContainsMouseChanged: {
    label.text = "wanghq" + model.display
}
```

因为 `label.text` 原来绑定到 `model.display`，你手动改 `label.text` 会覆盖绑定。

更推荐：

```qml
property bool debugPrefix: false

Label {
    id: label
    text: debugPrefix ? "wanghq" + model.display : model.display
}

onContainsMouseChanged: {
    debugPrefix = containsMouse
}
```

这时：

```text
debugPrefix 是状态；
model.display 是数据；
label.text 是由状态和数据推导出来的结果。
```

核心原则：

```text
不要直接改 UI 结果；
改状态；
让 UI 结果由绑定自动推导。
```

也就是：

```text
UI = f(状态, 数据)
```

---

## 8. 声明式 UI 的成熟心智模型

### 8.1 Excel 模型

```qml
width: parent.width / 2
```

像 Excel 公式：

```text
width = parent.width / 2
```

依赖变化时自动更新。

### 8.2 数据流模型

```qml
Rectangle {
    id: root
    width: 800

    Rectangle {
        id: box
        width: root.width / 2
    }

    Text {
        text: "box width = " + box.width
    }
}
```

依赖图是：

```text
root.width
   ↓
box.width
   ↓
text.text
```

读 QML 时要问：

```text
谁依赖谁？
谁变化会推动谁重新计算？
```

### 8.3 响应式编程模型

```qml
property bool hovered: false

color: hovered ? "blue" : "gray"

MouseArea {
    anchors.fill: parent
    onEntered: hovered = true
    onExited: hovered = false
}
```

事件只改变状态，UI 根据状态自动变化。

### 8.4 Single Source of Truth

单一事实来源的原则是：

```text
状态和数据是源头；
UI 属性是推导结果。
```

不要让多个地方同时维护同一个结果。

---

## 9. 对象、模板、实体：三者要区分

### 9.1 类型名是模板/类

```qml
Text
Rectangle
ListView
PlasmaCore.ToolTipArea
PlasmaComponents.Label
```

这些名字本身类似“类”或“构造器”，不是具体对象。

### 9.2 `{ ... }` 通常创建对象实体

```qml
Rectangle {
    id: box
    width: 100
    height: 50
}
```

这里创建了一个具体的 Rectangle 实体，`box` 指向这个实体。

### 9.3 `Component` 是显式模板

```qml
Component {
    id: redBoxTemplate

    Rectangle {
        width: 100
        height: 100
        color: "red"
    }
}
```

这里 `redBoxTemplate` 是模板，不是屏幕上的红色矩形实体。

需要通过 `Loader` 或 `createObject` 才能创建实体：

```qml
Loader {
    id: loader
    sourceComponent: redBoxTemplate
}
```

此时：

```text
loader 是 Loader 实体；
redBoxTemplate 是 Component 模板；
loader.sourceComponent 指向这个模板；
loader.item 是根据模板创建出的 Rectangle 实体。
```

---

## 10. `Loader { sourceComponent: redBoxTemplate }` 的准确含义

完整例子：

```qml
Component {
    id: redBoxTemplate

    Rectangle {
        width: 100
        height: 100
        color: "red"
    }
}

Loader {
    id: loader
    sourceComponent: redBoxTemplate
}
```

这句话：

```qml
sourceComponent: redBoxTemplate
```

的含义是：

```text
把 Loader 实体对象的 sourceComponent 属性设置为 redBoxTemplate 这个 Component 模板；
Loader 根据该模板创建真正的对象实体；
创建出来的实体可以通过 loader.item 访问。
```

注意：

```text
sourceComponent 需要的是 Component 模板。
mainItem 通常需要的是 Item 实体。
```

所以：

```qml
sourceComponent: redBoxTemplate
```

是：

```text
Component 属性 ← Component 模板
```

而：

```qml
mainItem: minimalToolTip
```

是：

```text
Item 属性 ← Item 实体
```

---

## 11. `mainItem: minimalToolTip` 的心智模型

你的 Plasma 代码类似：

```qml
PlasmaCore.ToolTipArea {
    id: toolTip

    mainItem: minimalToolTip

    PlasmaComponents.Label {
        id: minimalToolTip
        text: model.display
    }
}
```

这里：

```qml
mainItem: minimalToolTip
```

不是把文字赋给 `mainItem`，而是：

```text
toolTip.mainItem 指向 minimalToolTip 这个 Label 对象实体；
ToolTipArea 显示 tooltip 时，使用这个 Label 作为 tooltip 内容。
```

也就是说，真正显示的文字来自：

```qml
minimalToolTip.text
```

不是来自：

```qml
toolTip.mainText
```

如果已经设置了：

```qml
mainItem: minimalToolTip
```

那么修改 tooltip 文本时，更应该修改：

```qml
PlasmaComponents.Label {
    id: minimalToolTip
    text: "wanghq" + model.display
}
```

或者用状态控制：

```qml
property bool debugPrefix: false

PlasmaComponents.Label {
    id: minimalToolTip
    text: debugPrefix ? "wanghq" + model.display : model.display
}
```

---

## 12. Model-View-Delegate 模型

这是理解 `FolderItemDelegate.qml` 的关键。

```text
Model    = 数据
View     = 数据如何排列
Delegate = 每一项数据如何显示/编辑
```

简单例子：

```qml
ListView {
    model: ["A", "B", "C"]

    delegate: Text {
        text: modelData
    }
}
```

脑内模型：

```text
model 有 3 条数据；
delegate 是每一项的模板；
ListView 根据 delegate 创建 3 个 Text 实体。
```

运行时类似：

```qml
Text { text: "A" }
Text { text: "B" }
Text { text: "C" }
```

对应到 Plasma 桌面图标：

```text
FolderView / GridView
    负责排列桌面图标

Model
    桌面文件列表，每个文件有 display、size、type、blank 等字段

FolderItemDelegate.qml
    每一个桌面图标的显示模板
```

所以：

```qml
text: model.display
```

意思是：

```text
当前这个 delegate 实例对应的文件项，其 display 字段作为文本显示。
```

---

## 13. `model`、`modelData`、`delegate` 的来源

### 13.1 `ListView.model`

```qml
ListView {
    model: ["A", "B", "C"]
}
```

左边的 `model` 是 `ListView` 自带属性，不能随便改名。

错误：

```qml
ListView {
    myModel: ["A", "B", "C"]
}
```

### 13.2 数据源变量可以自定义

```qml
Item {
    property var names: ["A", "B", "C"]

    ListView {
        model: names

        delegate: Text {
            text: modelData
        }
    }
}
```

这里 `names` 是自己定义的属性名。

### 13.3 `modelData`

```qml
delegate: Text {
    text: modelData
}
```

`modelData` 是 Qt Quick 在 delegate 中自动提供的当前项数据。

对于：

```qml
model: ["A", "B", "C"]
```

分别是：

```text
第 0 项：modelData = "A"
第 1 项：modelData = "B"
第 2 项：modelData = "C"
```

### 13.4 `model.display`

在 Plasma 的 delegate 中：

```qml
text: model.display
```

这里：

```text
model       来自 delegate 机制
display     来自具体数据模型提供的数据角色
```

在 FolderView 场景中，`display`、`blank`、`size`、`type` 等字段通常来自 Plasma 的文件模型。

---

## 14. Delegate 是模板，不是单个对象

```qml
ListView {
    model: ["A", "B", "C"]

    delegate: Text {
        id: label
        text: modelData
    }
}
```

不要以为运行时只有一个 `label`。

真实情况是：

```text
第 0 项：
    label -> Text 实体，text = "A"

第 1 项：
    label -> Text 实体，text = "B"

第 2 项：
    label -> Text 实体，text = "C"
```

每个 delegate 实例里都有自己的 `label`。

这也解释了为什么 `FolderItemDelegate.qml` 不一定系统启动时全部执行。它是桌面图标的 delegate 模板，只有需要显示、刷新、hover、tooltip 时，Plasma 才会根据模板创建或更新具体实例。

---

## 15. 事件处理器才是写命令式代码的地方

错误写法：

```qml
Rectangle {
    width: 100
    console.log("hello")
}
```

QML 对象体里不能直接裸写 JS 语句。

正确写法：

```qml
Rectangle {
    width: 100

    Component.onCompleted: {
        console.log("hello")
    }
}
```

或者：

```qml
MouseArea {
    onClicked: {
        console.log("clicked")
    }
}
```

你之前遇到：

```text
Expected token ':'
```

就是因为在 QML 对象体内直接写了：

```qml
console.log(...)
```

解析器以为你要写属性：

```qml
console.log: ...
```

但没看到 `:`，所以报错。

---

## 16. 阅读 QML 的五类判断法

看到一行 QML，先判断它属于哪一类。

### 16.1 对象声明

```qml
Label {
    id: title
}
```

意思是：

```text
创建一个 Label 对象。
```

### 16.2 属性绑定

```qml
text: model.display
```

意思是：

```text
text 长期依赖 model.display。
```

### 16.3 对象引用

```qml
mainItem: minimalToolTip
```

意思是：

```text
mainItem 指向 minimalToolTip 这个对象。
```

### 16.4 事件处理器

```qml
onContainsMouseChanged: {
    console.log(containsMouse)
}
```

意思是：

```text
containsMouse 变化时执行这段 JS。
```

### 16.5 模板

```qml
delegate: Text {
    text: modelData
}
```

意思是：

```text
这是每一项的模板，View 会用它创建多个实体。
```

---

## 17. 结合 FolderItemDelegate.qml 的具体理解

典型片段：

```qml
PlasmaCore.ToolTipArea {
    id: toolTip

    active: plasmoid.configuration.toolTips && label.truncated && popupDialog === null
    interactive: false
    mainItem: minimalToolTip

    onContainsMouseChanged: {
        if (containsMouse && !model.blank) {
            main.GridView.view.hoveredItem = main
        }
    }

    PlasmaComponents.Label {
        id: minimalToolTip
        text: model.display
        textFormat: Text.PlainText
        wrapMode: Text.NoWrap
        elide: Text.ElideNone
    }
}
```

可以翻译成：

```text
创建一个 PlasmaCore.ToolTipArea，名字叫 toolTip。

toolTip.active 绑定到：
plasmoid.configuration.toolTips && label.truncated && popupDialog === null。

toolTip.mainItem 指向 minimalToolTip 这个 Label 实体。

创建一个 Label，名字叫 minimalToolTip。

minimalToolTip.text 绑定到当前文件项的 model.display。

鼠标进入或离开 ToolTipArea 时，执行 onContainsMouseChanged。
```

更推荐的调试写法：

```qml
property bool debugPrefix: false

PlasmaComponents.Label {
    id: minimalToolTip
    text: debugPrefix ? "wanghq" + model.display : model.display
}

onContainsMouseChanged: {
    debugPrefix = containsMouse
}
```

---

## 18. 历史脉络：MVC 到 Model-View-Delegate

Model-View-Delegate 不是像 MVC 那样由某一个人一次性提出的经典理论名称，它更像是 Qt/QML 对 MVC 思想的一种工程化变体。

大致脉络：

```text
MVC：Model-View-Controller
    早期源自 Smalltalk / Xerox PARC 相关工作。
    核心动机是把数据、显示、用户操作分离。

Qt Model/View
    把 View 和 Controller 的部分职责合并。
    引入 delegate 负责每一项的显示和编辑。

QML Model-View-Delegate
    model 提供数据；
    view 负责排列；
    delegate 负责每一项如何变成 UI。
```

动机可以总结为：

```text
把“数据是什么”“数据如何排列”“每一项长什么样、怎么交互”分开。
```

这样同一份数据可以用不同方式显示：

```text
图标视图
列表视图
表格视图
树视图
```

数据不需要跟着改变，只需要换 View 或 Delegate。

---

## 19. 最终总结

建立 QML 心智模型时，可以记住这几句话：

```text
1. QML 文件声明对象树，不是普通顺序脚本。
2. id 是当前作用域内的对象名，不是字符串。
3. 属性: 表达式 通常是绑定，不是一次性赋值。
4. 命令式赋值会覆盖绑定，所以尽量修改状态，不直接修改 UI 结果。
5. Component 和 delegate 是模板，Loader/View 根据模板创建实体。
6. Model-View-Delegate 中，model 是数据，view 是排列，delegate 是每项显示模板。
7. UI 应该被理解成状态和数据的函数：UI = f(state, data)。
```

如果用一句话压缩：

```text
QML 的核心不是“执行代码改变界面”，而是“声明数据、状态、对象之间的关系，让框架在变化时自动更新界面”。
```
