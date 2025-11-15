# Sky Mask Hole Filling

对天空分割结果进行二值化处理、连通域分析与自动填洞，生成干净的天空掩码（0/1）

---

## 项目简介

本脚本用于对 **天空语义分割结果（sky mask）** 进行后处理，通过阈值分割、形态学操作、连通域检测等步骤，自动识别并**填补天空区域中的黑洞（holes）**，生成平滑连续的天空区域掩码。

适用于：

* DeepLab、U-Net 等模型输出的天空 mask 后处理
* SVF（Sky View Factor）计算前的图像净化
* 城市街景天空可视化
* 道路或建筑轮廓识别时的天空区域补全

---

## 功能与特性

* 自动二值化 & 形态学闭运算
* 通过连通域检测找出 **背景"黑洞"**
* 仅保留从图像边界连通的真实“背景区域”
* 自动填补被包围的天空小洞
* 输入 RGB 图，输出 0/1（保存为 0/255 图像）
* 支持批量处理整个文件夹

---

## 使用方法

### 1. 安装依赖

```bash
pip install opencv-python numpy
```

### 2. 修改脚本配置

在 `__main__` 部分设置路径：

```python
input_dir = r'D:\to_zero\result\yl_mask'
output_dir = r'D:\to_zero\result\filled'
```

### 3. 运行脚本

```bash
python fill_sky_holes.py
```

处理后的 mask 输出为：

* 黑底白字（0/255）
* 洞已自动填补
* 可直接用于 SVF / CNN 计算

---

## 原理说明

### 1. 二值化 + 形态学闭运算

从 RGB 转灰度，根据亮度大致分出天空：

```python
_, sky_mask = cv2.threshold(gray, 200, 1, cv2.THRESH_BINARY)
```

闭运算用于：

* 平滑天空边缘
* 避免小黑洞噪点

---

### 2. 找出真正"背景"区域

通过连通域分析：

```python
connectedComponentsWithStats
```

仅保留与图像边界连通的连通域（真实背景），其余都视为“被包围的小洞”。

---

### 3. 填洞

黑洞 = 背景 - 边界背景
最终 mask：

```python
filled = sky + holes
```

---

## 输入输出说明

### 输入（RGB 图）

* 天空为亮色
* 建筑/障碍物较暗
* 包含模型输出的粗 mask 或原图均可

### 输出（单通道）

* 0：非天空
* 255：天空

适合作为：
✔ SVF 计算输入
✔ 二次语义分割 mask
✔ 建筑遮挡分析

---

## 示例处理流程

```
原始 RGB
    │
灰度 & 二值化
    │
形态学闭运算（清除小缝隙）
    │
背景连通域分析
    │
识别“黑洞”
    │
填补天空区域的小洞
    │
输出干净的天空 mask
```

---

## 核心函数描述

### `pre_process_sky(img)`

二值化天空 + 形态学闭运算
返回 0/1 mask

### `keep_border_cc(mask)`

保留从边缘连通的连通域
用于区分背景 vs 真正“洞”

### `fill_sky_holes(img)`




