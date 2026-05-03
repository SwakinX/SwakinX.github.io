---
title: "PySide PyQt多语言国际化翻译工作流以及相关问题"
description: "PySide PyQt使用tr()方法的一些注意事项和一个多语言国际化翻译工作流分享"
published: 2026-05-01
tags: ["PySide6", "Qt", "Python", "GUI", "workflow", "translation"]
category: 'Share'
draft: false 
---
# 正确标记文本
这里默认你已经了解了国际化的流程和工具，具体可以[参考](https://www.hawu.me/dev/3417)，[C++版](https://runebook.dev/zh/docs/qt/qcoreapplication/translate)更底层的解释
## 使用.tr()
只要是使用.tr()标记的文本不管.tr()在这里能否直接访问，都会被`pyside6-lupdate`工具（PyQt5是`pylupdate5`）添加到.ts文件。

对于类本身就继承自`QObject`的（QT组件都是），可以直接使用`self.tr()`标记文本，对于类没有继承`QObject`的，使用别的继承了`QObject`的对象调用.tr()虽然可以正常标记文本，但想要不修改文件.ts直接使用就必须让类继承`QObject`然后调用`self.tr()`。

因为生成的.ts里`context`名称默认为类名，使用其他对象或类的.tr()方法都不能正确处理`context`名称，要自己去修改成正确的名称才能正常生效非常麻烦。直接使用`QObject.tr()`或`QCoreApplication.tr()`都是不行的，这时`context`名称直接为空，无法对应上。
## 使用QCoreApplication.translate()
```python
QApplication.translate(context: str, sourceText: str, disambiguation: str = None, n: int = -1) -> str
```

|参数           |说明                                                               |
| ------------- | ----------------------------------------------------------------- |
|context        |上下文名称，通常用类名（如 'InfoBar' ），用于区分不同模块中相同的源文本 |
|sourceText     |要翻译的源文本（如 '提示' ）                                         |
|disambiguation |（可选）消歧字符串，当同一上下文中有相同源文本但不同含义时使用          |
|n              |（可选）用于处理复数形式， -1 表示不使用复数                          |

对于无法直接调用或继承QObject的情况，直接使用`QApplication.translate()`，一般只需要传入前两项参数，虽然这样很长，但不要改写名称例如`tr=QApplication.translate`或封装方法后使用，这样`pyside6-lupdate`工具要么会错误理解内容要么无法识别到标记文本。
## 使用自定义的翻译器管理
如果不想每次都使用一长串来标记，可以放弃qt的自动托管，使用自定的翻译管理类。思路就是在需要标记的地方实例化一个`QObject`对象，然后调用.tr()进行标记，这样`pyside6-lupdate`工具会将对象名作为`context`名称，需要使用对应文本时可以通过`context`名称和原始文本从 qm 文件返回翻译文本。这里只提供简单的思路，我还未在自己的项目中实现。
# 翻译工作流说明

## 翻译管理脚本

让AI写一个脚本例如 `update_i18n.py`，具备以下功能，主要根据项目文件创建.ts文件、将.ts文件中要翻译的文本到处为JSON，将JSON文件扔给免费AI补全翻译文本，再用脚本将翻译好的内容应用到.ts文件中，最后编译成.qm文件。

为什么不直接让AI翻译.ts文件，因为文件结构特殊，AI也是要先提取文本翻译了再生成文件的，这样效率低而且很容易漏掉某些文本，不如直接给结构化的文本，AI直接一对一自动翻译，校对翻译内容时直接看JSON文件也更方便，当然不用JSON，用表格也是可以的。

### 命令说明

```bash
# 初始化创建 .ts 文件
python devTools/update_i18n.py --init

# 更新翻译源文件
python devTools/update_i18n.py --update

# 导出待翻译文本到 JSON
python devTools/update_i18n.py --export

# 导出所有翻译（包括已完成的）
python devTools/update_i18n.py --export-all

# 从 JSON 文件读取翻译并应用到 .ts
python devTools/update_i18n.py --apply

# 编译翻译文件
python devTools/update_i18n.py --release

# 执行完整工作流（更新 + 编译）
python devTools/update_i18n.py --all

# 查看帮助
python devTools/update_i18n.py --help
```

### 标准翻译工作流

```bash
# 1. 更新翻译源文件
python devTools/update_i18n.py --update

# 2. 导出所有翻译到 JSON 以便编辑
python devTools/update_i18n.py --export-all

# 3. 编辑 translations.json，添加/修改翻译

# 4. 应用翻译到 .ts 文件
python devTools/update_i18n.py --apply

# 5. 编译为 .qm 文件
python devTools/update_i18n.py --release
```

### 仅添加新翻译的工作流

```bash
# 1. 更新翻译源文件
python devTools/update_i18n.py --update

# 2. 只导出未完成的翻译
python devTools/update_i18n.py --export

# 3. 在 translations.json 中为未完成的条目添加翻译

# 4. 应用翻译
python devTools/update_i18n.py --apply

# 5. 编译
python devTools/update_i18n.py --release
```

## 参数详解

| 参数         | 别名 | 说明                                             |
| ------------ | ---- | ------------------------------------------------ |
| --init       | -i   | 初始化创建 .ts 文件                              |
| --update     | -u   | 从 Python/UI 代码提取可翻译字符串，更新 .ts 文件 |
| --release    | -r   | 将 .ts 编译为二进制 .qm 文件                     |
| --all        | -a   | 同时执行 --update 和 --release（默认行为）       |
| --export     | -e   | 导出未完成的翻译条目到 JSON（直接覆盖）          |
| --export-all |      | 导出所有翻译条目到 JSON（直接覆盖）              |
| --apply      |      | 从 translations.json 读取翻译并更新 .ts 文件     |
| --help       | -h   | 显示帮助信息                                     |

## translations.json 文件格式

新的 JSON 格式是一条文本对应多个语言版本，更加直观：

```json
[
  {
    "source": "源文本（可选）",
    "comment": "注释文本（用于匹配，优先）",
    "zh_CN": "简体中文翻译",
    "zh_TW": "繁体中文翻译",
    "en_US": "英文翻译",
    "ja_JP": "日文翻译"
  },
  {
    "comment": "警告",
    "zh_CN": "警告",
    "en_US": "Warning"
  }
]
```

### 格式说明

- **source**: 原始文本（可选，用于匹配）
- **comment**: 注释文本（可选，用于匹配，优先级高于 source）
- **zh_CN**, **zh_TW**, **en_US**, **ja_JP**: 各语言的翻译内容

匹配规则：先匹配 comment，没有则匹配 source。
