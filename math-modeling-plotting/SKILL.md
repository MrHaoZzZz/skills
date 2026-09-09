---
name: math-modeling-plotting
description: 数学建模竞赛科研绘图(MATLAB/Python)国奖级美化技巧：雷达图、3D柱状图、3D饼图、折线图、灵敏度分析置信区间图等。当用户要"画图""绘图""可视化""科研图表""美化图表""雷达图/柱状图/饼图"时使用。
---

# 数学建模国奖级科研绘图

产出竞赛级（国奖级）科研图表的核心方法与 MATLAB 代码范式。

## 一、AI 驱动绘图工作流

1. 先发题目 + 相关文献给 AI，让它阅读学习领域知识，并告诉你求解需要哪些数据集。
2. 逐问求解，数据用绝对路径传给 AI，让其借助 MATLAB 代码完成。
3. 每张图生成后追问：「这是借助你的代码绘制出的图像，你认为图像结果合理么？如果合理，请用一段 250 字的自然段进行解释（可以结合文献）。如果不合理，请指出不合理之处，并修改代码。」
4. 结果不理想时，可直接要求：「根据阅读的文献和对赛题的理解，直接生成比较合理的结果图像」。

## 二、常用图表类型与范式

**权重雷达图 / 多边形雷达图**
- 用 `theta = pi/2 : -2*pi/num_vars : -3*pi/2` 从 12 点方向顺时针排布。
- `fill` 绘制背景多边形与数据覆盖多边形（浅色填充 + 深色描边，`FaceAlpha` 控制透明度）。
- 内部虚线多边形网格 + 从中心向外的轴线（Spokes）。
- 标签用 `'Interpreter','latex'` 还原数学公式斜体字效（如 `$PM_{2.5}$`、`$AQI$`）。
- 按象限自动调整文字对齐（`HorizontalAlignment`/`VerticalAlignment`）。

**3D 渐变柱状图**
- `bar3(Z)` 后逐柱设置 `h(k).CData = h(k).ZData`（颜色映射为高度）、`FaceColor='interp'`、`EdgeColor='none'`。
- 自定义 colormap：定义关键 RGB 锚点 + `interp1` 插值生成 256 级平滑色带（逆向 Spectral 风格）。
- 加柱子顶部数值标签、Times New Roman 字体、`view(-40,25)` + `material('dull')` + `camlight('left')` + `lighting gouraud` 增强立体阴影。

**3D 饼图**
- `pie3(data, explode, labels)`，用 `colormap(colors_pie)` 自定义每块颜色（RGB/255 归一化）。
- 白色边框贴合质感、`view(-30,45)` + `camlight left` + `lighting flat`。
- 图例用不可见 `fill([NaN NaN NaN],...)` 补丁生成句柄以精确控制顺序与颜色。

**折线图（多曲线）**
- 用 `patch` 绘制专业加分项：如大气透明窗口阴影（8-13 μm），`FaceAlpha` 半透明 + 文字标注。
- 字体统一 Times New Roman、坐标轴带单位（`\mu m`）、图例标题化（`title(lgd,'Film Thickness')`）。
- 导出高清图：`exportgraphics(gcf,'x.png','Resolution',600)`。

**灵敏度分析图（置信区间）**
- 蒙特卡洛模拟（如 PDMS 厚度工艺误差→净制冷功率），X 轴为随机干扰幅度，Y 为结果。
- `fill([x, fliplr(x)], [CI_Upper, fliplr(CI_Lower)], ...)` 绘制 95% 置信区间阴影。
- 均值线用 `plot(...,'-o',...)` 加白心圆点标记；可加注释箭头标注「Highly Robust Region」。

## 三、图表通用规范

- 必备：标准标题、坐标轴带单位、图例、30-50 字专业图注。
- 字体：Times New Roman（轴/标签/图例），LaTeX 渲染数学符号。
- 避免单一色调，可加辅助符号、误差线、阴影区域。
- 配色美观、有专业加分项（阴影窗口、置信区间、光照立体感）。
