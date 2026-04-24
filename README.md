# Tianjic A Graph Software

## 环境准备

建议使用 VS Code 作为主要开发环境。

### 创建并激活 Conda 环境
```bash
conda create -n tianjic_a_software python=3.10
conda activate tianjic_a_software
pip install -r requirements.txt
# 安装pip包，Pytorch可能无法直接安装，requirements.txt中有安装命令
# ubuntu上安装graphviz方法
sudo apt install grahpviz
```
[不同操作系统下graphviz安装方法](https://graphviz.org/download/)

### VS Code 配置提示

仓库已经在 [.vscode/launch.json](.vscode/launch.json) 中配置了 `PYTHONPATH` 与 `cwd`，调试时会自动使用项目根目录作为工作路径；此外， [.vscode/settings.json](.vscode/settings.json) 已将项目根目录加入 `python.analysis.extraPaths`，便于 Pylance 解析本地模块。若使用其他 IDE，请自行确保同样的路径设置。

## 快速运行

1. 在 VS Code 中将打开的文件夹定位于仓库根目录（即包含 `basics/`、`core_components/` 等目录的层级）。
2. 安装 Python 扩展与调试器后，打开任意测试脚本（例如 [tests/maxpooling_test.py](tests/maxpooling_test.py) 或 [tests/resnet8_test.py](tests/resnet8_test.py)）。
3. 使用 `Ctrl+F5` 直接运行，或按 `F5` 进入调试模式，脚本正常执行即表示环境就绪。

## 项目结构
```
root
├── basics/                # 基础数据结构与工具函数
├── core_components/       # 图表示、调度、编译等核心组件
├── operations/            # 高层运算节点定义
├── prims/                 # 原语实现与参数生成
├── test_gen/              # 测例生成脚本
├── tests/                 # 集成与单元测试
├── requirements.txt       # Python 依赖列表
└── README.md              # 本文件
```

## 开发说明

### 软件目标与运行流程概述

- **执行单元**：接收原语（256-bit PIC）及数据，按照原语指令从内存读取参数与输入，计算后写回内存。
- **芯片控制逻辑**：依次从内存读取原语并下发给执行单元，直到遇到停机原语结束。
- **软件职责**：生成执行所需的原语代码、参数以及对应输入/输出数据，供硬件仿真或测试平台使用。

### 关键目录与模块

- 基础数据类型与内存抽象：[`core_components/data.py`](core_components/data.py)
- 图构建与硬件映射：[`core_components/hardware_graph.py`](core_components/hardware_graph.py)、[`core_components/graph.py`](core_components/graph.py)
- 原语与参数生成：[`prims`](prims/) 模块，模板可参考 [`prims/template.py`](prims/template.py)
- 运算节点定义：[`operations`](operations/) 目录

### 添加新原语流程

1. 在 [`prims`](prims/) 下创建新文件，可参考模板 [`prims/template.py`](prims/template.py)。
2. 根据硬件设计更新原语参数打包逻辑与地址分配。
3. 如需高层封装，在 [`operations`](operations/) 中新增对应 Operation，并实现 `infer`、`para_node`、`gen_prim` 等方法。
4. 在相关测试或生成脚本中新增样例，确保通过已有测试框架执行。

> **注意**：请及时同步硬件原语定义的最新参数与寄存器布局，以免生成内容与实际设计不一致。