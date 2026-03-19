# 设计文档

## 概述

本设计解决 Streamlit 应用中 Plotly 图表ID冲突问题，通过实现智能的key生成策略和统一的图表渲染管理，确保每个图表都有唯一的标识符。

## 架构

解决方案将通过以下方式实现：
1. 创建统一的图表key生成函数
2. 识别所有 `st.plotly_chart()` 调用位置
3. 为每个图表调用添加唯一的key参数
4. 实现基于上下文的key生成策略

## 组件和接口

### 图表Key生成器

```python
def generate_chart_key(context: str, chart_type: str = None, 
                      data_hash: str = None, position: str = None) -> str:
    """为图表生成唯一的key标识符"""

def get_data_hash(df: pd.DataFrame) -> str:
    """生成数据的哈希值用于key生成"""

def create_chart_with_key(fig, context: str, **kwargs) -> None:
    """带有自动key生成的图表渲染函数"""
```

### 上下文管理器

```python
class ChartContext:
    """图表上下文管理器，跟踪图表的来源和位置"""
    
    def __init__(self, source: str, location: str):
        self.source = source  # 'history', 'new_query', 'sidebar'
        self.location = location  # 'main_chart', 'trend_chart', etc.
        
    def get_key_prefix(self) -> str:
        """获取key前缀"""
```

## 数据模型

### 图表标识符配置
```python
@dataclass
class ChartKeyConfig:
    context: str
    chart_type: Optional[str]
    data_source: str
    position: str
    timestamp: Optional[str]
    
    def generate_key(self) -> str:
        """生成唯一的图表key"""
```

## 正确性属性

*属性是系统在所有有效执行中都应该保持为真的特征或行为——本质上是关于系统应该做什么的正式声明。属性是人类可读规范和机器可验证正确性保证之间的桥梁。*

### 属性1：Key唯一性
*对于任何* 在同一页面上显示的图表集合，生成的key应该都是唯一的，不会产生冲突
**验证需求：需求1.1, 1.2, 1.3**

### 属性2：Key一致性
*对于任何* 相同上下文和数据的图表，在多次渲染时应该生成相同的key
**验证需求：需求1.3, 2.4**

### 属性3：功能保持性
*对于任何* 添加了key参数的图表，其交互功能和响应式行为应该与原来完全相同
**验证需求：需求2.1, 2.2, 2.3**

### 属性4：上下文区分性
*对于任何* 来自不同上下文（历史消息vs新查询）的图表，应该生成不同的key前缀
**验证需求：需求1.5, 3.3, 3.4**

## 错误处理

### Key冲突检测
- 实现key冲突检测机制
- 在检测到潜在冲突时自动添加后缀
- 记录key生成过程用于调试

### 降级策略
- 当key生成失败时，使用时间戳作为后备方案
- 提供手动key指定的选项
- 确保即使在异常情况下图表也能正常显示

### 调试支持
- 添加详细的日志记录key生成过程
- 提供key冲突的诊断信息
- 支持开发模式下的key可视化

## 测试策略

### 单元测试
- 测试key生成函数的唯一性
- 测试不同上下文下的key区分
- 测试边界情况和异常处理

### 属性测试
- 测试key唯一性在随机数据下的保持
- 测试key一致性在重复渲染下的保持
- 测试功能完整性在添加key后的保持

### 集成测试
- 测试完整的图表渲染流程
- 测试多图表页面的ID冲突解决
- 测试历史消息和新查询的图表共存