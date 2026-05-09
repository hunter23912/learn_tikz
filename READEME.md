# TikZ 模型图项目说明

这个项目用于用 LaTeX + TikZ 绘制 EEG 迁移学习相关模型结构图。当前主要模型包括 DAN-MMD、DANN、MSMDA 和 DMMR。

## 项目结构

```text
learn_tikz/
├── main.tex                  # 主入口：选择要绘制的完整模型
├── run.sh                    # 常用编译和导出命令
├── models/                   # 完整模型图，只负责组装模块
│   ├── dan_mmd.tex           # DAN-MMD 模型图
│   ├── dann.tex              # DANN 模型图
│   ├── msmda.tex             # MSMDA 多源域适应模型图
│   └── dmmr.tex              # DMMR 模型图
└── modules/                  # 可复用组件和样式
    ├── model_common.tex      # 通用节点、箭头、图例
    ├── eeg_signal.tex        # EEG 数据卡片和波形
    ├── cfe.tex               # CFE 特征压缩模块
    ├── cube.tex              # 伪 3D 特征片
    ├── domain_feature.tex    # 源域/目标域特征点云
    ├── msmda_components.tex  # MSMDA 专用组件
    └── dmmr_components.tex   # DMMR 专用组件
```

基本思想是：

- `main.tex` 只作为入口，选择一个模型。
- `models/` 放完整模型结构，只做模块拼接和布局。
- `modules/` 放可复用组件，比如 EEG 输入卡片、CFE、特征点云、通用图例、通用箭头样式。
- 相同的视觉元素尽量抽到 `modules/`，避免每个模型重复写一遍。

## 切换要绘制的模型

在 `main.tex` 中只保留一个 `\input{...}` 处于启用状态：

```tex
% \input{models/dan_mmd.tex} % 绘制DAN-MMD模型
% \input{models/dann.tex}    % 绘制DANN模型
% \input{models/msmda.tex}   % 绘制MSMDA模型
\input{models/dmmr.tex}      % 绘制DMMR模型
```

想画哪个模型，就取消对应那一行的注释，其余模型保持注释。

## TikZ 图的基本组成

### tikzpicture

一张 TikZ 图通常写在：

```tex
\begin{tikzpicture}
  % 这里写节点、连线、模块
\end{tikzpicture}
```

可以把它理解成“画布”。模型文件里的主要绘图内容都放在这个环境中。

### tikzset

`\tikzset{...}` 用来集中定义样式和可复用模块。

```tex
\tikzset{
  dann edge/.style={
    -{Latex[length=1.5mm]}, % 箭头样式
    draw=teal!50!black,    % 连线颜色
    line width=.55pt       % 连线宽度
  }
}
```

之后就可以直接写：

```tex
\draw[dann edge] (a) -- (b);
```

这样所有同类箭头都能统一修改。

### node

`\node` 用来画一个节点，可以是矩形、圆形、文字、模块框。

```tex
\node[dann head] (emoHead) at (0, 0) {Linear};
```

含义：

- `dann head`：使用的样式。
- `(emoHead)`：节点名字，后续可以用它连线。
- `at (0, 0)`：节点位置。
- `{Linear}`：节点中显示的文字。

常见连接锚点：

```tex
emoHead.west   % 节点左侧
emoHead.east   % 节点右侧
emoHead.north  % 节点上方
emoHead.south  % 节点下方
```

### draw

`\draw` 用来画线、箭头、边框。

```tex
\draw[dann edge] (source-east) -- (cfe-west);
```

常见线段写法：

```tex
(a) -- (b)       % 直线
(a) |- (b)       % 先竖后横的折线
(a) -| (b)       % 先横后竖的折线
```

如果要让线连接到节点边界，优先使用节点锚点，而不是写死坐标。

### pic

`pic` 是 TikZ 中的“可复用小模块”。比如项目中的 CFE、EEG 波形、domain feature 都是 pic。

定义方式大致是：

```tex
\tikzset{
  pics/example module/.style={
    code={
      \node[...] (box) at (0, 0) {Module};
    }
  }
}
```

使用方式：

```tex
\pic at (2, 0) {example module};
```

这个项目中推荐：

- 单个可复用视觉组件写成 `pic` 或宏，放入 `modules/`。
- 完整模型只在 `models/` 中引用这些组件。

### scope

`scope` 用来临时改变局部坐标、缩放或图层。

```tex
\begin{scope}[shift={(2, 0)}, scale=.5]
  \pic {cfe module};
\end{scope}
```

含义：

- `shift={(2, 0)}`：整体平移。
- `scale=.5`：整体缩小到 50%。

常见背景层写法：

```tex
\begin{scope}[on background layer]
  \node[fit=(a)(b)(c)] (box) {};
\end{scope}
```

这通常用来画模块外框，避免外框盖住内部节点。

### fit

`fit` 可以让一个节点自动包住其他节点。

```tex
\node[
  rounded corners=4pt,
  draw=gray,
  fit=(nodeA)(nodeB)(nodeC),
  inner sep=3mm
] (groupBox) {};
```

它适合做“区域外框”。注意：如果一个子区域本身有内边距，也要把代表这个子区域实际大小的 fit 节点放进外层 fit 中，否则可能出现子区域撑出外框的问题。

## 当前项目中的关键公共模块

### EEG 输入卡片

定义在：

```text
modules/eeg_signal.tex
```

常用方式：

```tex
\eegDataCard{sourceData}{Source data}{dann source data}
```

三个参数分别是：

- 节点名字。
- 显示标题。
- 使用的样式。

### CFE 模块

定义在：

```text
modules/cfe.tex
```

使用方式：

```tex
\pic {cfe module};
```

内部用四个伪 3D 特征片表示 `310 -> 256 -> 128 -> 64` 的特征压缩过程。

### 源域/目标域特征点云

定义在：

```text
modules/domain_feature.tex
```

常用模块：

```tex
\pic {source feature cloud};  % 源域特征
\pic {target feature cloud};  % 目标域特征
\pic {domain feature pair};   % 上下两个域特征 + 右侧花括号
```

### 通用图例

定义在：

```text
modules/model_common.tex
```

使用方式：

```tex
\domainFeatureLegend{modelBox}{6mm}
```

含义：

- `modelBox`：图例挂在哪个区域下方。
- `6mm`：图例距离该区域的垂直距离。

## 编译为 PDF

推荐使用：

```bash
latexmk -pdf main.tex
```

生成：

```text
main.pdf
```

如果只是用 LaTeX Workshop，保存后它会根据 VS Code 设置调用 `latexmk`。

## 导出为 SVG

如果要用 `dvisvgm` 导出 SVG，需要先生成 `.dvi`：

```bash
latex -interaction=nonstopmode main.tex
dvisvgm main.dvi -n -o output.svg
```

例如导出 DANN：

```bash
latex -interaction=nonstopmode main.tex
dvisvgm main.dvi -n -o dann.svg
```

参数说明：

- `latex`：生成 `main.dvi`。
- `dvisvgm main.dvi`：把 DVI 转为 SVG。
- `-n`：不把文字转换成路径，方便后续编辑文字。
- `-o dann.svg`：指定输出文件名。

如果 SVG 在其他软件中字体显示不一致，可以去掉 `-n`，让文字转成路径：

```bash
dvisvgm main.dvi -o dann.svg
```

## 常用维护建议

- 新模型放到 `models/`，例如 `models/new_model.tex`。
- 新的可复用组件放到 `modules/`。
- 模型之间不要互相 `\input`，公共部分提取到 `modules/`。
- 连线尽量使用节点锚点，比如 `.east`、`.west`，少写死坐标。
- 外框优先使用 `fit=(...)` 自动包围内容。
- 大块区域建议分成独立 `scope` 或独立 pic，避免一个区域的坐标变化影响其他区域。
- 样式集中写进 `\tikzset`，不要在每个 `\node` 里重复堆样式。
