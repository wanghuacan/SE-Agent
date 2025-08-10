# SE Framework - SWE-agent 多样性实验框架

## 概述

SE框架是基于SWE-agent的轻量级扩展，专注于对同一问题执行多次不同的解决尝试，通过策略编排和算子系统实现解决方案的多样性。

> **注意**: 本文档专门介绍 SE 框架的详细设计和使用。如需了解整个项目架构和其他框架（Atlas等），请参考根目录的 [CLAUDE.md](/CLAUDE.md)。

## 🚀 快速入门

### 第一次使用？从这里开始！

#### 1️⃣ 运行交互式演示 (推荐)
```bash
# 快速体验 (5分钟)
python SE/demo_data_flow.py --mode quick

# 详细了解 (15分钟) 
python SE/demo_data_flow.py --mode detailed

# 开发者模式 (30分钟)
python SE/demo_data_flow.py --mode developer
```

#### 2️⃣ 运行测试套件
```bash
# 验证系统完整性
python SE/test/run_operator_tests.py

# 测试特定算子
python SE/test/test_alternative_strategy.py
```

#### 3️⃣ 阅读学习指南
```bash
📚 SE/LEARNING_GUIDE.md    # 完整学习路径
📖 本文档下方的 "📊 核心数据格式规范" 部分
```

#### 4️⃣ 运行实际实验
```bash
# 演示模式（不执行真实SWE-agent）
python SE/basic_run.py --mode demo

# 执行模式（需要配置API key）
python SE/basic_run.py --mode execute --config SE/configs/se_configs/test_deepseek_se.yaml
```

## 核心设计理念

### 🎯 目标
对每个问题进行多次迭代，确保每次都产生真正不同的解决路径和策略。

### 🏗️ 架构层次
```
SE_Runner (外层统一管理器)
    ↓ 管理全局配置 (output_dir, model, api等)  
    ↓ 执行策略编排 (第1次、第2次、第3次...)
    ↓ 
    ↓ 循环多次迭代:
    ↓   ├── 算子处理历史数据 (如需要)
    ↓   ├── 生成局部参数
    ↓   └── 调用SWE_Iterator
    ↓
SWE_Iterator (单次运行器)
    ↓ 接收全局参数 + 局部参数
    ↓ 执行单次SWE-agent运行
    ↓ 输出到 output_dir/instance_name/{迭代编号}/
```

### 🔄 参数传递设计
- **全局参数**: 由SE_Runner管理，固定不变 (model, output_dir, api配置等)
- **局部参数**: 由算子过程输出决定 (instance_templates_dir, --enhance-history-filter-json)

## 项目结构

```
SE/
├── run.py                          # SE框架主入口（空文件）
├── basic_run.py                    # 实际可用的SE基础运行器
├── operator_dev.py                 # 算子开发测试脚本
├── configs/                        # 配置文件目录
│   ├── se_configs/                 # SE主配置文件
│   │   ├── example_experiment.yaml
│   │   └── test_deepseek_se.yaml
│   ├── base_configs/               # SWE-agent基础配置
│   │   ├── baseconfig1.yaml
│   │   └── baseconfig2.yaml
│   └── test_config/                # 测试配置
│       └── test_claude.yaml
├── core/                           # 核心管理模块
│   ├── __init__.py                 # 核心模块接口（空）
│   ├── swe_iterator.py             # SWE-agent统一迭代运行器
│   └── utils/                      # 工具函数目录
│       ├── __init__.py
│       ├── se_logger.py            # SE框架日志系统
│       └── organize_folder.py      # 工作目录格式转换工具
├── operators/                      # 算子系统
│   ├── __init__.py                 # 算子统一接口
│   ├── base.py                     # 算子基类定义
│   └── registry.py                 # 算子注册管理系统
├── instances/                      # 测试实例目录
│   ├── test.json                   # 测试实例配置
│   ├── 1.json, 5.json, 8.json...  # 具体测试用例
│   └── caogao                      # 草稿文件夹
├── test/                           # 测试文件目录
│   ├── README.md
│   ├── api_test.py
│   ├── converter_old.py            # 旧版轨迹转换器参考
│   └── counter_generation/         # 计数器生成测试
└── trajectories/                   # SE框架运行时输出目录
    ├── Demo_Structure/             # 标准输出结构示例
    │   ├── iteration_1/            # 第一次迭代结果
    │   ├── iteration_2/            # 第二次迭代结果
    │   └── se_framework.log        # SE框架日志
    └── operator_dev_test/          # 算子开发测试输出
        └── test_20250714_xxxxx/    # 时间戳命名的测试运行
```

## 参数编写指南

### 工作目录规范
- **项目根目录**: `/home/uaih3k9x/630_swe` 
- **SE框架位置**: `SE/` (相对于根目录)
- **运行方式**: 所有SE相关命令必须在根目录执行
- **路径格式**: 配置文件中所有路径相对于根目录编写

> **重要**: SE框架作为整个项目的一个子模块，严格遵循项目级别的路径和配置约定。

### 配置参数流转
1. **iterator_config对象**: 每次迭代中可变的参数
   - `base_config`: 基础配置名称
   - `special_arguments`: 算子输出的参数（json文件/文件夹路径）,指定到`instance_templates_dir`或`--enhance-history-filter-json`

2. **路径编写原则**:
   ```yaml
   # 正确：相对于根目录
   json_file: "SE/instances/5.json"
   output_dir: "SE/trajectories/testt_5"
   
   # 错误：绝对路径或相对路径混淆
   json_file: "/home/uaih3k9x/630_swe/SE/instances/5.json"
   output_dir: "./trajectories/testt_5"
   ```

3. **SWE Iterator调用格式**:
   ```bash
   # 标准调用格式
   python SE/core/swe_iterator.py SE/configs/base_configs/test_claude.yaml
   
   # 带增强功能的调用
   python SE/core/swe_iterator.py SE/configs/base_configs/test_claude.yaml --enhance-history-filter-json path/to/enhancement.json
   ```

## Utils

### 日志系统 (`se_logger.py`)

- **文件路径**: `SE/core/utils/se_logger.py`
- **功能**: 基于SWE-agent现有日志系统，为SE框架提供统一的日志管理

#### 主要特性
1. **文件存储**: 日志保存在每次运行的output_dir下（如 `SE/trajectories/testt_5/se_framework.log`）
2. **全级别记录**: 记录DEBUG、INFO、WARNING、ERROR所有级别的信息
3. **模块化设计**: 支持不同模块使用不同emoji区分（⚙️ 📋 🔧 🎯 等）
4. **线程安全**: 基于SWE-agent的线程安全日志系统

#### 基本使用
```python
from SE.core.utils.se_logger import setup_se_logging, get_se_logger

# 1. 设置日志系统（在运行开始时调用一次）
log_file = setup_se_logging("SE/trajectories/testt_5")

# 2. 获取logger实例（在各个模块中使用）
logger = get_se_logger("basic_run", emoji="⚙️")

# 3. 记录日志
logger.info("SE基础运行脚本启动")
logger.debug("详细调试信息")
logger.warning("警告信息")
logger.error("错误信息", exc_info=True)  # 包含异常堆栈
```

#### 日志文件格式
```
2025-07-13 10:30:15,123 - INFO - SE.basic_run - SE基础运行脚本启动
2025-07-13 10:30:15,124 - DEBUG - SE.basic_run - 使用配置文件: SE/configs/se_configs/test_deepseek_se.yaml
2025-07-13 10:30:15,125 - ERROR - SE.basic_run - 运行出错: FileNotFoundError
```

### 工作目录转换工具 (`organize_folder.py`)

- **文件路径**: `SE/core/utils/organize_folder.py`
- **功能**: 将工作目录从iterator（SWE-agent）风格转换为SE风格，以控制迭代的稳健性

#### 主要功能
1. **目录结构转换**: 将`trajectory/instance_id/files`格式转换为`trajectory/instance_id/1/files`的迭代运行结构
2. **轨迹文件处理**: 自动处理`.traj`文件，生成简化的`.tra`文件用于分析
3. **内容截断优化**: 对长内容进行智能截断，减少存储空间和提高处理效率
4. **统计信息记录**: 生成处理日志，记录token数量和文件处理统计

#### 设计参考
- 基于`test/converter_old.py`的逻辑设计（仅作参考，需根据SE框架需求重新实现）
- 支持迭代式组织，确保多次运行不冲突
- 提供轨迹文件的标准化处理流程

#### 使用场景
- 将SWE-agent原始输出转换为SE框架兼容格式
- 清理和优化轨迹文件存储
- 为后续分析和处理提供标准化数据结构

> **注意**: 该工具当前为规划阶段，具体实现需要考虑SE框架的具体需求和数据流转规范。


## 配置系统

### 工作目录规范
**项目根目录**: `/home/uaih3k9x/630_swe`  
**SE框架运行**: 从根目录执行SE相关命令  
**SE Iterator调用格式**: `python SE/core/swe_iterator.py SE/configs/base_configs/test_claude.yaml`

> **路径约定**: 所有配置文件路径相对于项目根目录编写，确保与项目整体架构一致。

### SE主配置文件 (se_configs/*.yaml)
```yaml
# SE框架的全局配置，由SE_Runner使用
# 运行目录: /home/uaih3k9x/630_swe (根目录)

# 主模型配置 (SWE-agent使用，遵循LiteLLM格式)
model:
  name: "anthropic/claude-sonnet-4-20250514"
  api_base: "your_api_base"
  api_key: "your-api-key"
  max_input_tokens: 128000
  max_output_tokens: 64000
  per_instance_cost_limit: 0.01    # SWE必要参数
  total_cost_limit: 0.1             # SWE必要参数

# 算子模型配置 (算子处理时使用，可选)
operator_models:
  name: "anthropic/claude-sonnet-4-20250514"
  api_base: "your_api_base"
  api_key: "your-api-key"
  max_input_tokens: 128000
  max_output_tokens: 64000
  # 注意：不需要cost limit参数

# 实例配置 (与现有格式完全一致)
instances:
  json_file: "SE/instances/test.json"
  key: "instances"
  subset: "verified"
  split: "test"
  shuffle: false
  evaluate: false

# 输出配置 (重要参数)
output_dir: "SE/trajectories/experiment_001"  # 最重要参数之一
suffix: "specific_instances"
num_workers: 14

# 策略编排：定义多次迭代的执行计划
strategy:
  iterations:
    - base_config: "test_claude"           # 第1次：基础配置
      operator: null
    - base_config: "baseconfig1"           # 第2次：不同配置
      operator: null  
    - base_config: "test_claude"           # 第3次：基础配置+算子
      operator: "Crossover"
```

### SWE-agent基础配置 (base_configs/*.yaml)
```yaml
# 纯净的SWE-agent配置，给SWE_Iterator使用
agent:
  model: 
    name: "anthropic/claude-sonnet-4-20250514"
tools:
  - name: "bash"
  - name: "str_replace_editor"
# ... 其他SWE-agent原生配置
```

## 算子系统

算子是**数据处理流程**，基于现代化的类架构设计。每个算子：

1. **输入**: 工作目录下的历史数据（.traj轨迹文件等）
2. **处理**: 分析轨迹数据，提取问题陈述，调用LLM生成策略
3. **输出**: 符合`instance_templates_dir`或`enhance_history_filter_json`格式的文件/路径
4. **传递**: 将生成的路径参数传给SWE_Iterator

### 算子架构设计

#### 基础架构
- **BaseOperator**: 所有算子的抽象基类，提供通用功能
  - LLM模型管理和API调用
  - 轨迹数据加载和问题陈述提取  
  - 并发实例处理框架
  - 日志记录和错误处理

- **TemplateOperator**: 模板算子基类
  - 生成系统提示模板（YAML格式）
  - 返回`instance_templates_dir`参数
  - 支持策略前缀自定义

- **FilterOperator**: 过滤算子基类（规划中）
  - 生成历史过滤配置（JSON格式）
  - 返回`enhance_history_filter_json`参数

#### 注册管理系统
- **动态注册**: 通过`register_operator()`注册算子类
- **运行时创建**: 通过`create_operator()`创建算子实例
- **类型检查**: 确保所有算子继承自BaseOperator
- **全局注册表**: 统一管理所有可用算子

### 开发中算子

#### DemoTemplateOperator (演示算子)
- **类型**: TemplateOperator
- **功能**: 演示算子系统的基本功能
- **策略前缀**: "DEMO SOLUTION STRATEGY"
- **用途**: 开发测试和功能验证

### 规划中算子

#### Crossover (交叉对比算子)
- **功能**: 基于历史轨迹生成对比策略
- **输出**: `instance_templates_dir` 路径
- **策略**: 分析历史尝试，生成差异化的系统提示

#### Conclusion (结论推导算子)  
- **功能**: 专注于解决方案的收敛
- **输出**: `enhance_history_filter_json` 路径
- **策略**: 智能指导，帮助收敛到最终解决方案

#### SummaryBug (错误总结算子)
- **功能**: 基于错误模式生成避免策略
- **输出**: `instance_templates_dir` 路径  
- **策略**: 分析历史错误，生成错误避免提示

### 算子开发

#### 开发测试
使用`operator_dev.py`进行算子开发和测试：
```bash
# 运行算子开发测试
python SE/operator_dev.py

# 支持的功能：
# - 模拟轨迹环境创建
# - 算子注册验证
# - 基础功能测试
# - 错误处理验证
```

#### 创建新算子
```python
from SE.operators import TemplateOperator, register_operator
from SE.core.utils import get_problem_description

class MyCustomOperator(TemplateOperator):
    def get_name(self):
        return "my_custom"
    
    def get_strategy_prefix(self):
        return "MY CUSTOM STRATEGY"
    
    def _generate_content(self, instance_info, problem_description, trajectory_data):
        # 实现具体的生成逻辑
        return "生成的策略内容"

# 注册算子
register_operator("my_custom", MyCustomOperator)
```

## 数据流设计

```
SE配置文件 
    ↓
SE_Runner (解析strategy字段)
    ↓
循环每个iteration:
    ↓
算子处理 (如需要)
    ↓ 输出局部参数
传递全局参数+局部参数
    ↓
SWE_Iterator (执行单次运行)
    ↓ 输出到 iteration_{num}/
```

## 实际执行功能

### basic_run.py 实现方案A

`SE/basic_run.py` 已实现完整的多迭代执行功能，采用**方案A: 直接调用swe_iterator.py**：

#### 核心机制
1. **临时配置生成**: 为每次迭代动态生成临时YAML配置文件
2. **subprocess调用**: 通过`subprocess.run()`调用`SE/core/swe_iterator.py`
3. **自动清理**: 执行完成后自动删除临时配置文件

#### 使用方式
```bash
# 演示模式（查看流程，不执行SWE-agent）
python SE/basic_run.py --mode demo

# 直接执行模式（实际运行SWE-agent）
python SE/basic_run.py --mode execute

# 指定配置文件
python SE/basic_run.py --config SE/configs/se_configs/custom.yaml --mode execute
```

#### 输出结构
每次运行生成唯一时间戳目录，采用标准的多迭代结构：
```
SE/trajectories/Demo_Structure/
├── iteration_1/
│   ├── preds.json                          # 预测结果汇总
│   ├── run_batch.config.yaml               # 运行配置记录
│   ├── run_batch.log                       # 批处理日志
│   ├── run_batch_exit_statuses.yaml        # 退出状态记录
│   ├── sphinx-doc__sphinx-8548/             # 实例目录
│   │   ├── sphinx-doc__sphinx-8548.config.yaml  # 实例配置
│   │   ├── sphinx-doc__sphinx-8548.debug.log    # 调试日志
│   │   ├── sphinx-doc__sphinx-8548.info.log     # 信息日志
│   │   ├── sphinx-doc__sphinx-8548.pred         # 预测结果
│   │   ├── sphinx-doc__sphinx-8548.trace.log    # 跟踪日志
│   │   └── sphinx-doc__sphinx-8548.traj         # 轨迹文件
│   ├── sphinx-doc__sphinx-8551/             # 其他实例...
│   └── ... (更多实例目录)
├── iteration_2/
│   ├── preds.json                          # 第二次迭代结果
│   ├── run_batch.config.yaml
│   ├── run_batch.log
│   ├── run_batch_exit_statuses.yaml
│   ├── sphinx-doc__sphinx-8548/             # 相同实例的第二次尝试
│   │   ├── sphinx-doc__sphinx-8548.config.yaml
│   │   ├── sphinx-doc__sphinx-8548.debug.log
│   │   ├── sphinx-doc__sphinx-8548.info.log
│   │   ├── sphinx-doc__sphinx-8548.pred
│   │   ├── sphinx-doc__sphinx-8548.trace.log
│   │   └── sphinx-doc__sphinx-8548.traj
│   └── ... (相同实例的不同迭代)
└── se_framework.log                        # SE框架总体日志
```

#### 核心特性

**🎯 双模式运行支持**
- `--mode demo`: 演示模式，显示配置流程但不执行SWE-agent
- `--mode execute`: 直接执行模式，自动运行所有迭代

**⚙️ 动态配置管理**
- **时间戳目录**: 每次运行生成唯一的`test_{timestamp}`目录，避免冲突
- **临时配置生成**: 为每次迭代动态创建临时YAML配置文件
- **参数实时显示**: 执行前显示实际传递给SWE-agent的所有参数
- **自动清理**: 执行完成后自动删除临时配置文件

**🔄 策略驱动的多迭代执行**
- **配置驱动**: 从SE配置文件的`strategy.iterations`读取迭代计划
- **三参数变化设计**: 每次迭代仅有3个关键参数发生变化
  - `base_config`: 每次迭代使用不同的SWE-agent基础配置
  - `operator`: 每次迭代可能使用不同的算子策略
  - `output_dir`: 每次迭代独立的输出目录
- **算子参数扩展**: 当operator有返回值时，会动态添加额外参数
  - `instance_templates_dir`: Crossover等算子返回的模板目录路径
  - `enhance_history_filter_json`: Conclusion等算子返回的JSON文件路径
- **参数继承**: 全局参数（model、instances、num_workers等）在迭代间保持一致

**📋 完整的日志和监控系统**
- **SE框架日志**: 记录到`se_framework.log`，包含详细的执行流程
- **SWE-agent输出**: 终端完整显示SWE-agent的思考和执行过程
- **配置透明**: 显示传递给每次迭代的实际配置参数
- **状态追踪**: 实时显示每次迭代的成功/失败状态

**🛡️ 错误处理和恢复机制**
- **异常捕获**: 完善的异常处理，记录详细错误信息
- **失败处理**: 某次迭代失败时可选择停止或继续
- **资源清理**: 确保临时文件和资源得到正确清理
- **日志记录**: 所有错误和异常都记录到日志文件

## 核心术语

### 运行 (Run)
- **定义**: SE系统的一次完整执行
- **包含**: 多个迭代(iterations)的完整过程
- **输出路径**: `SE/trajectories/{唯一ID}/`

### 迭代 (Iteration)
- **定义**: 一次SWE-agent的具体执行
- **表现**: 实例目录下的`iteration_{num}/`文件夹 (iteration_1/, iteration_2/, iteration_3/, ...)
- **内容**: 包含`.traj`, `.patch`, `.pred`, `preds.json`等SWE-agent输出文件

### 算子 (Operator)
- **定义**: 数据处理流程，不是具体的代码文件
- **用途**: 分析历史数据，生成运行时增强参数
- **示例**: `Crossover`, `Conclusion`, `SummaryBug`

### 策略编排
- **定义**: 通过算子组合实现的运行策略
- **示例**: `config1+null`, `config2+null`, `config1+Crossover`

## 使用方式

### 工作目录设置
```bash
# 确保在项目根目录运行所有SE相关命令
cd /home/uaih3k9x/630_swe
```

> **注意**: SE框架与项目中的其他框架（Atlas等）共享同一工作目录约定。

### SWE Iterator调用格式 (当前可用)
```bash
# 标准格式：使用@符号引用配置文件
python SE/swe_iterator.py @SE/configs/base_configs/test_claude.yaml

# 或者直接路径 (相对于根目录)
python SE/swe_iterator.py SE/configs/test_config/test_claude.yaml
```

### SE_Runner用法 (待实现)
```bash
# 使用SE主配置执行完整实验
python SE/SE_runner.py SE/configs/se_configs/example_experiment.yaml

# 使用特定的se_config
python SE/SE_runner.py SE/configs/se_configs/test_claude_se.yaml
```

### 算子测试
```python
# 测试算子功能
from SE.operators import get_operator

op = get_operator("Crossover")
templates_dir, json_file = op.process(workspace_dir, instances)
```

## 实现状态

### ✅ 已完成
- [x] **目录结构清理**: 删除过度复杂的抽象层
- [x] **配置系统重组**: se_configs/ 和 base_configs/ 分离
- [x] **算子系统架构**: 完整的基类和注册系统（operators/base.py, registry.py）
- [x] **SWE_Iterator实现**: 统一迭代运行器（core/swe_iterator.py）
- [x] **SE日志系统**: 基于SWE-agent的专用日志管理（core/utils/se_logger.py）
- [x] **基础运行器**: 可用的多迭代执行系统（basic_run.py）✨
- [x] **算子开发工具**: 开发测试脚本（operator_dev.py）
- [x] **标准输出结构**: Demo_Structure展示的多迭代目录组织
- [x] **实例测试数据**: 测试用例配置文件（instances/目录）
- [x] **轨迹处理系统**: 完整的.tra文件生成和token压缩功能 ✨
- [x] **Problem提取功能**: 自动提取problem描述为.problem文件 ✨
- [x] **开发指南文档**: Python路径问题解决方案和最佳实践 ✨

### 🚧 进行中
- [ ] **具体算子实现**: Crossover、Conclusion、SummaryBug算子
- [ ] **SE_Runner**: 外层统一管理器（基于basic_run.py扩展）
- [ ] **完整工作流测试**: 端到端功能验证

### 📋 架构完成度
- **核心架构**: ✅ 完成 - 算子基类、注册系统、日志系统
- **运行系统**: ✅ 完成 - 迭代运行器、基础执行器、轨迹处理
- **配置管理**: ✅ 完成 - 多层配置系统
- **开发工具**: ✅ 完成 - 开发测试脚本、命令行工具
- **数据处理**: ✅ 完成 - .tra生成、problem提取、token统计
- **具体算子**: 🚧 进行中 - 需要实现具体的算子逻辑

## 与其他框架的关系

SE框架作为整个 630_swe 项目的一个组件，与其他框架的关系如下：

- **基于SWE-agent**: 使用 `sweagent/` 目录中的核心功能，通过Hook机制实现干预
- **继承统一运行器**: SWE_Iterator基于项目中已有的统一运行器设计模式
- **独立实验层**: SE框架专注于多样性实验，不修改其他框架的底层逻辑
- **数据兼容**: 可以处理和分析来自其他框架的轨迹数据

> **架构定位**: SE框架是项目的实验扩展层，专注于多迭代策略而非替换现有框架。

## 开发原则

1. **简单性**: 避免过度抽象，专注SE框架核心功能
2. **可扩展性**: 算子系统支持新策略的轻松添加
3. **清晰职责**: SE_Runner管理全局，SWE_Iterator负责执行，算子处理数据
4. **项目兼容**: 保持与整个 630_swe 项目生态的兼容性

## 重要澄清和风险点

### 术语澄清

#### 根目录概念
- **根目录**: `/home/uaih3k9x/630_swe` 指项目根目录，所有git操作在此执行
- **路径编写**: 配置文件中路径均相对于根目录，便于版本控制和部署

#### Operator设计原理
- **输入**: SWE-agent产生的轨迹数据和历史文件
- **处理**: 分析、生成个性化策略信息
- **输出**: 返回文件夹路径或JSON文件路径（不是直接的参数值）
- **消费**: SWE_Iterator根据special_arguments在运行时按需读取路径内容


---

*本文档既是SE框架的开发指南，也是功能需求规范，为开发和使用提供统一参考。*

## 🔥 最新更新（2025-07-14）

### 🎯 **轨迹处理系统完成**

实现了完整的轨迹文件处理和数据提取功能：

#### **核心功能**
- **自动.tra文件生成**: 每次iteration后自动压缩轨迹文件
- **智能Token压缩**: 平均压缩率75%-87%，大幅节省存储和处理成本
- **Problem描述提取**: 自动提取`<pr_description>`标签内容为`.problem`文件
- **详细统计日志**: 记录原始token数、压缩后token数、节省比例等详细信息

#### **压缩效果示例**
```
🎬 INFO 已创建 sphinx-doc__sphinx-9461.tra: 112 历史项, 6258 tokens 
        (原始: 48622, 节省: 42364, 压缩率: 87.1%)
```

#### **使用方式**
```bash
# 自动集成：basic_run.py执行时自动生成
python SE/basic_run.py --mode execute

# 手动生成：独立工具
python SE/core/utils/generate_tra_files.py workspace_dir --extract-problems

# 只提取problem文件
python SE/core/utils/generate_tra_files.py workspace_dir --problems-only
```

#### **输出结构增强**
```
iteration_1/
├── sphinx-doc__sphinx-8548/
│   ├── sphinx-doc__sphinx-8548.traj     # 原始轨迹（完整）
│   ├── sphinx-doc__sphinx-8548.tra      # 简化轨迹（压缩80%+）✨
│   ├── sphinx-doc__sphinx-8548.problem  # 问题描述（纯文本）✨
│   ├── sphinx-doc__sphinx-8548.pred
│   └── ...其他文件
```

### 🛠️ **开发体验改进**

#### **Python路径问题彻底解决**
- 创建了完整的开发指南（`SE/DEVELOPMENT_GUIDE.md`）
- 标准化了模块导入模式
- 实现了延迟导入机制避免循环导入
- 提供了路径问题调试方法

#### **basic_run.py完全可用**
- 修复了所有模块导入错误
- 支持演示模式和执行模式
- 自动集成轨迹处理功能
- 完整的错误处理和日志记录

### 📊 **数据处理能力**

#### **Token压缩技术**
- **智能截断**: 保留内容的前20%和后10%关键信息
- **角色区分**: assistant保留thought/action，其他角色保留content
- **工具优化**: 对str_replace_editor等长输出特别优化
- **统计精确**: 提供详细的压缩前后对比数据

#### **问题描述获取精度**
- **统一接口**: `get_problem_description()` 提供标准化获取方式
- **多源支持**: 自动选择.problem文件、轨迹提取或JSON配置的最佳来源
- **优先级管理**: .problem文件 > 轨迹提取 > JSON配置
- **验证机制**: `validate_problem_availability()` 检查所有可用获取方法
- **准确定位**: 自动识别`Trajectory[1]["content"][0]["text"]`位置
- **标签解析**: 精确提取`<pr_description>`标签内容
- **格式清理**: 输出纯净的问题描述文本
- **批量处理**: 支持workspace级别的批量提取

### 📝 **问题描述管理系统** ✨ **NEW**

SE框架提供了统一的问题描述获取和管理接口，解决了多源数据不一致的问题：

#### **统一接口设计**
```python
from SE.core.utils import get_problem_description, validate_problem_availability

# 智能获取 - 自动选择最佳来源
problem = get_problem_description('path/to/instance')

# 指定来源 - 精确控制获取方式
problem_from_file = get_problem_description('instance_path', method='file')
problem_from_traj = get_problem_description('instance_path', method='trajectory') 
problem_from_json = get_problem_description('instance_path', method='json')

# 验证可用性 - 检查所有获取方法
validation = validate_problem_availability('instance_path')
```

#### **多源获取策略**
1. **`.problem`文件** (最高优先级)
   - 纯文本格式，直接可用
   - 由轨迹处理系统自动生成
   - 适合算子快速读取

2. **轨迹文件提取** (中等优先级)
   - 从`.traj`或`.tra`文件解析`<pr_description>`标签
   - 实时提取，保证数据最新
   - 自动处理复杂的JSON结构

3. **JSON配置** (备用选项)
   - 从实例配置文件获取
   - 支持静态问题描述
   - 待实现功能

#### **验证结果示例**
```json
{
  "instance_name": "sphinx-doc__sphinx-10435",
  "methods_available": ["file", "trajectory"],
  "primary_source": "file",
  "problem_length": 245,
  "problem_preview": "Incorrect result with Quaterniont.to_rotation_matrix()..."
}
```

### 🔧 **工具链完善**

#### **命令行工具**
- `generate_tra_files.py`: 功能完整的轨迹处理工具
- 支持dry-run模式预览
- 支持单iteration和workspace两种处理模式
- 支持.tra和.problem文件的独立或联合生成

#### **Python API**
```python
from SE.core.utils import TrajectoryProcessor, extract_problems_from_workspace
from SE.core.utils import get_problem_description, validate_problem_availability

# 轨迹处理
processor = TrajectoryProcessor()
stats = processor.process_iteration_directory(Path('iteration_1'))

# 问题描述获取 (统一接口)
problem = get_problem_description('path/to/instance')  # 自动选择最佳来源
problem = get_problem_description('path/to/instance', method='file')  # 优先.problem文件
problem = get_problem_description('path/to/instance', method='trajectory')  # 从轨迹提取

# 问题描述验证
validation = validate_problem_availability('path/to/instance')
# 返回: {"methods_available": ["file", "trajectory"], "primary_source": "file", ...}

# 问题描述批量提取
results = extract_problems_from_workspace('workspace_dir')
```

### 🤖 **LLM集成轨迹分析系统** ✨ **NEW**

SE框架现在支持智能LLM驱动的轨迹分析和总结功能：

#### **核心功能**
- **智能轨迹总结**: 使用LLM深度分析每次迭代的解决方案方法
- **结构化输出**: 提取关键信息如修改文件、技术策略、推理模式等
- **自动备用机制**: LLM调用失败时自动降级到基础总结
- **轨迹池管理**: 多迭代实验的完整轨迹数据管理和分析

#### **LLM分析输出示例**
```json
{
  "approach_summary": "修复LaTeX输出中不需要的空白，通过修改LaTeX writer使用%标记防止内联代码块周围的空格插入",
  "modified_files": ["/testbed/sphinx/writers/latex.py", "/testbed/test_latex_inline_code.py"],
  "key_changes": ["添加%标记到sphinxcode等内联代码包装器", "修改LaTeXTranslator中所有相关方法"],
  "strategy": "1. 识别问题 2. 定位代码 3. 应用LaTeX空白控制 4. 验证修改",
  "specific_techniques": ["LaTeX空白控制", "测试驱动验证"],
  "reasoning_pattern": "问题识别→代码调查→解决方案设计→实现验证"
}
```

#### **自动集成**
- **无缝集成**: SE框架自动使用LLM分析每次迭代轨迹
- **配置灵活**: 支持operator_models配置独立于主模型
- **智能降级**: LLM不可用时自动使用基础总结保证系统稳定性

#### **轨迹池增强**
```
trajectories/test_{timestamp}/
├── iteration_1/                    # 迭代执行结果
├── iteration_2/                    # 迭代执行结果
└── traj.pool                       # 🆕 智能分析的轨迹总结池
```

#### **测试验证**
- **集成测试**: `test/llm_integration/` 目录包含完整测试用例
- **真实数据验证**: 已在实际轨迹数据上验证LLM分析质量
- **性能优化**: 智能Token管理和API调用优化

### 📋 **可立即使用功能**

1. **完整的多迭代SE框架**: `python SE/basic_run.py --mode execute`
2. **轨迹数据自动压缩**: 每次iteration自动生成.tra文件  
3. **🆕 问题描述统一获取**: 标准化的问题描述获取和验证接口
4. **🆕 智能轨迹分析**: LLM驱动的深度轨迹理解和总结
5. **详细的性能统计**: 压缩率、token节省等关键指标
6. **健壮的错误处理**: 解决了所有已知的Python路径问题

### 🎯 **下一步计划**

- **具体算子实现**: 基于完善的.tra、.problem和LLM分析数据实现Crossover、Conclusion等算子
- **算子生态扩展**: 利用LLM分析结果开发更智能的算子类型
- **性能优化**: 基于压缩数据和LLM分析的高效算子处理流程
- **多模型支持**: 扩展对更多LLM提供商的支持

## 📊 **核心数据格式规范**

SE框架为operator提供了统一的数据访问接口，确保数据流转的标准化和一致性。

### **四种核心数据格式**

#### **1. Problem Description (问题描述)**
- **定义**: 每个实例的问题陈述，描述需要解决的具体问题
- **格式**: 纯文本字符串
- **获取方式**: 
  - 优先级1: `.problem`文件 (由轨迹处理系统自动生成)
  - 优先级2: 从轨迹文件解析`<pr_description>`标签
  - 优先级3: JSON配置文件中的描述
- **用途**: 为operator提供问题上下文，生成针对性策略

#### **2. TRA文件 (压缩轨迹)**
- **定义**: 经过智能压缩的轨迹数据，保留关键信息
- **格式**: JSON格式，包含简化的执行步骤和结果
- **特点**: 
  - 平均压缩率75%-87%
  - 保留关键的思考过程和执行动作
  - 去除冗余的中间输出
- **用途**: 为operator提供高效的历史执行数据分析

#### **3. PATCH文件 (预测结果)**
- **定义**: 实例的最终预测结果，通常为代码修改或解决方案
- **格式**: 文本格式，通常为diff格式或纯文本
- **文件优先级**: `.patch` > `.pred` (重要更新)
- **用途**: 展示每次迭代的具体输出和解决方案

#### **4. Trajectory Pool (轨迹池)**
- **定义**: 实例在多个迭代中的智能分析总结数据
- **格式**: JSON格式，包含LLM分析的结构化洞察
- **包含信息**:
  - 问题描述
  - 每次迭代的方法总结
  - 修改文件列表
  - 技术策略和推理模式
- **用途**: 为operator提供跨迭代的历史分析和趋势洞察

### **统一数据访问接口**

#### **基础使用**
```python
from SE.core.utils.instance_data_manager import get_instance_data_manager

# 获取数据管理器
manager = get_instance_data_manager()

# 获取单个实例的完整数据
instance_data = manager.get_instance_data(instance_path)

# 访问四种核心数据
problem = instance_data.problem_description  # 问题描述
tra_content = instance_data.tra_content      # 压缩轨迹
patch_content = instance_data.patch_content  # 预测结果 (.patch优先)
traj_content = instance_data.traj_content    # 原始轨迹 (可选)
```

#### **轨迹池访问**
```python
# 获取实例在轨迹池中的完整数据
pool_data = manager.get_traj_pool_data(traj_pool_path, instance_name)

# 获取特定迭代的总结
iteration_summary = manager.get_instance_iteration_summary(
    traj_pool_path, instance_name, iteration_number
)
```

#### **批量处理**
```python
# 获取整个迭代目录的所有实例
instances = manager.get_iteration_instances(iteration_dir)

# 数据完整性验证
for instance in instances:
    completeness = manager.validate_instance_completeness(instance)
    print(f"实例 {instance.instance_name}: {completeness['completeness_score']}% 完整")
```

### **Operator开发标准模式**

```python
from SE.core.utils.instance_data_manager import get_instance_data_manager

class MyOperator(TemplateOperator):
    def _generate_content(self, instance_info, problem_description, trajectory_data):
        manager = get_instance_data_manager()
        
        # 1. 获取当前实例数据
        instance_data = manager.get_instance_data(instance_info['path'])
        
        # 2. 访问标准化数据
        problem = instance_data.problem_description
        tra_data = instance_data.tra_content
        patch_data = instance_data.patch_content
        
        # 3. 获取轨迹池历史数据
        pool_data = manager.get_traj_pool_data(
            self.traj_pool_path, instance_info['name']
        )
        
        # 4. 分析和生成策略
        strategy = self.analyze_and_generate_strategy(
            problem, tra_data, patch_data, pool_data
        )
        
        return strategy
```

### **数据完整性保证**

#### **优先级读取机制**
- **PATCH文件**: `.patch` → `.pred` (自动选择最佳)
- **问题描述**: `.problem` → 轨迹提取 → JSON配置
- **轨迹数据**: `.tra` → `.traj` (压缩优先)

#### **验证和调试**
```python
# 验证实例数据完整性
validation = manager.validate_instance_completeness(instance_data)
print(f"完整性: {validation['completeness_score']}%")
print(f"缺失数据: {validation['missing_data']}")

# 检查可用文件
print(f"可用文件: {instance_data.available_files}")
```

### **向后兼容性**

SE框架保持与现有代码的兼容性，同时提供新的标准化接口：

```python
# 旧式接口 (仍然支持)
from SE.core.utils.traj_extractor import TrajExtractor
extractor = TrajExtractor()
legacy_data = extractor.extract_instance_data(iteration_dir)

# 新式接口 (推荐)
structured_data = extractor.extract_instances_structured(iteration_dir)
```

### **最佳实践**

1. **总是使用统一接口**: 通过`InstanceDataManager`访问数据
2. **验证数据完整性**: 使用`validate_instance_completeness()`检查
3. **处理缺失数据**: 优雅处理`None`值和不完整数据
4. **利用轨迹池**: 充分利用跨迭代的历史分析数据
5. **遵循优先级**: 信任系统的文件优先级选择机制

这套数据格式规范确保了SE框架中operator的数据访问一致性，为扩展和维护提供了坚实的基础。