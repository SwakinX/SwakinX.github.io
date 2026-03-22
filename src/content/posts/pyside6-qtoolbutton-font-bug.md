---
title: "PySide6 QToolButton 字体Bug：setPixelSize在6.10.1+版本中的问题"
description: "详细分析PySide6 6.10.1+版本中QToolButton使用setPixelSize或样式表font-size时出现的字体错误，提供完整的复现代码和解决方案"
published: 2026-03-22
tags: ["PySide6", "Qt", "Python", "GUI", "Bug", "QToolButton"]
category: 'Share'
draft: false 
---

# PySide6 QToolButton 字体Bug分析

## 问题描述

在PySide6 6.10.1+版本中，当`QToolButton`使用`QFont.setPixelSize`方法设置字体或在样式表中包含`font-size`设置时，鼠标离开按钮时会触发以下错误：

```
QFont::setPointSize: Point size <= 0 (-1), must be greater than 0
```

## 影响范围

- **版本**：仅影响PySide6 6.10.1+版本
- **组件**：仅影响`QToolButton`
- **触发条件**：
  - 使用`setPixelSize`设置字体
  - 或在样式表中设置`font-size`
  - 鼠标悬停后离开按钮

## 复现步骤

1. 创建使用`setPixelSize`的`QToolButton`
2. 应用包含`font-size`设置的样式表
3. 鼠标悬停在按钮上
4. 鼠标移出时出现错误

## 完整测试代码

```python
"""
PySide6 QToolButton Font Bug Test
Demonstrates the font issue with QToolButton in PySide6 6.10.1+
"""
import sys
import PySide6
from PySide6.QtCore import Qt
from PySide6.QtGui import QFont
from PySide6.QtWidgets import QApplication, QWidget, QVBoxLayout, QToolButton, QLabel


class SimpleTestWidget(QWidget):
    """ Simple test interface for PySide6 font bug """
    
    def __init__(self, parent=None):
        super().__init__(parent)
        self.setup_ui()
        
    def setup_ui(self):
        layout = QVBoxLayout(self)
        layout.setSpacing(20)
        
        # Title with version info
        title = QLabel("PySide6 Version: " + PySide6.__version__ + " - QToolButton Font Bug Test")
        title.setAlignment(Qt.AlignCenter)
        title.setStyleSheet("font-size: 16px; font-weight: bold;")
        layout.addWidget(title)
        
        # QToolButton with setPixelSize (no stylesheet)
        pixel_button_no_style = QToolButton(self)
        pixel_button_no_style.setText("setPixelSize(14) - No Stylesheet")
        
        font_pixel = QFont()
        font_pixel.setFamilies(['Segoe UI', 'Microsoft YaHei'])
        font_pixel.setPixelSize(14)  # Using setPixelSize
        pixel_button_no_style.setFont(font_pixel)
        
        layout.addWidget(pixel_button_no_style)
        
        # QToolButton with setPixelSize (with stylesheet) - BUG APPEARS HERE
        pixel_button_with_style = QToolButton(self)
        pixel_button_with_style.setText("setPixelSize(14) - With Stylesheet (BUG)")
        
        font_pixel2 = QFont()
        font_pixel2.setFamilies(['Segoe UI', 'Microsoft YaHei'])
        font_pixel2.setPixelSize(14)  # Using setPixelSize
        pixel_button_with_style.setFont(font_pixel2)
        
        # Apply stylesheet (simulates the bug condition)
        style_sheet = """
        QToolButton {
            background: rgba(255, 255, 255, 0.7);
            border: 1px solid rgba(0, 0, 0, 0.073);
            border-radius: 5px;
            padding: 5px 12px 6px 12px;
        }
        """
        pixel_button_with_style.setStyleSheet(style_sheet)
        
        layout.addWidget(pixel_button_with_style)
        
        # QToolButton with setPointSize (with stylesheet) - WORKAROUND
        point_button_with_style = QToolButton(self)
        point_button_with_style.setText("setPointSize(10) - With Stylesheet (Workaround)")
        
        font_point = QFont()
        font_point.setFamilies(['Segoe UI', 'Microsoft YaHei'])
        font_point.setPointSize(10)  # Using setPointSize (workaround)
        point_button_with_style.setFont(font_point)
        
        point_button_with_style.setStyleSheet(style_sheet)
        layout.addWidget(point_button_with_style)

        style_sheet2 = """
        QToolButton {
            background: rgba(255, 255, 255, 0.7);
            border: 1px solid rgba(0, 0, 0, 0.073);
            border-radius: 5px;
            padding: 5px 12px 6px 12px;
            font-size: 14px;
        }
        """
        point_button_with_style2 = QToolButton(self)
        point_button_with_style2.setText("setPointSize(10) - With Stylesheet and set Font-size (bug)")
        
        font_point2 = QFont()
        font_point2.setFamilies(['Segoe UI', 'Microsoft YaHei'])
        font_point2.setPointSize(10)  
        point_button_with_style2.setFont(font_point2)
        
        point_button_with_style2.setStyleSheet(style_sheet2)
        
        layout.addWidget(point_button_with_style2)  
        # Instructions
        desc = QLabel("Hover mouse over buttons to test the font bug.")
        desc.setStyleSheet("color: gray; font-size: 12px;")
        layout.addWidget(desc)
        
        self.resize(500, 350)

if __name__ == "__main__":
    app = QApplication(sys.argv)
    window = SimpleTestWidget()
    window.setWindowTitle("PySide6 QToolButton Font Bug Test")
    window.show()
    sys.exit(app.exec())
```

## 临时解决方案

### 方案1：使用setPointSize替代setPixelSize

```python
# 不安全的代码
font = QFont()
font.setPixelSize(14)  # 会触发bug

# 安全的代码
font = QFont()
font.setPointSize(10)  # 使用setPointSize作为替代
```

### 方案2：避免在样式表中设置font-size

```css
/* 不安全的样式表 */
QToolButton {
    font-size: 14px;  /* 会触发bug */
}

/* 安全的样式表 */
QToolButton {
    /* 移除font-size设置 */
}
```

## 技术分析

### 问题根源

这个bug似乎与PySide6 6.10.1+版本中字体处理逻辑的变化有关。当鼠标离开`QToolButton`时，样式表的状态切换过程中，字体大小被错误地设置为-1，触发了Qt的字体验证机制。

### 组件特异性

为什么只影响`QToolButton`？：

- `QToolButton`通常用于工具栏，可能有不同的样式表处理逻辑

## 社区反馈

由于Qt官方的bug跟踪系统需要商业权限，目前无法直接报告此bug。建议开发者：

1. 使用本文提供的临时解决方案
2. 在社区论坛中分享此问题
3. 关注PySide6的后续版本更新

## 结论

虽然这个bug似乎没啥影响，只会在控制台出现提醒，但通过使用`setPointSize`替代`setPixelSize`和优化样式表设置或使用低于6.10.1版本的包，可以完全避免这个问题。

---

*本文档基于实际测试和代码分析，适用于PySide6 6.10.1+版本。如有更新或修复，请关注PySide6官方发布说明。*