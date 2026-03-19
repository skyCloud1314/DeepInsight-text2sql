# 设计文档

## 概述

本设计实现了一个智能的错误上下文传递系统，在模型重试时将上一次的错误信息集成到新的prompt中，帮助模型进行更精准的修正。

## 架构

解决方案包含以下核心组件：
1. 错误信息收集器 - 捕获和标准化错误信息
2. 错误上下文管理器 - 管理错误历史和上下文
3. Prompt增强器 - 将错误信息集成到重试prompt中
4. 重试策略管理器 - 基于错误模式优化重试策略

## 组件和接口

### 错误信息收集器

```python
class ErrorCollector:
    """错误信息收集和标准化"""
    
    def capture_exception(self, exception: Exception, context: dict) -> ErrorInfo:
        """捕获异常信息"""
    
    def capture_execution_error(self, command: str, output: str, 
                               exit_code: int) -> ErrorInfo:
        """捕获执行错误"""
    
    def capture_timeout_error(self, operation: str, timeout: float) -> ErrorInfo:
        """捕获超时错误"""
```

### 错误上下文管理器

```python
class ErrorContextManager:
    """管理错误历史和上下文"""
    
    def add_error(self, error_info: ErrorInfo) -> None:
        """添加错误信息到历史"""
    
    def get_retry_context(self, max_errors: int = 3) -> RetryContext:
        """获取重试上下文"""
    
    def analyze_error_patterns(self) -> List[ErrorPattern]:
        """分析错误模式"""
```

### Prompt增强器

```python
class PromptEnhancer:
    """将错误信息集成到prompt中"""
    
    def enhance_retry_prompt(self, original_prompt: str, 
                           retry_context: RetryContext) -> str:
        """增强重试prompt"""
    
    def format_error_context(self, errors: List[ErrorInfo]) -> str:
        """格式化错误上下文"""
    
    def summarize_long_errors(self, error_text: str, max_length: int) -> str:
        """摘要过长的错误信息"""
```

## 数据模型

### 错误信息结构
```python
@dataclass
class ErrorInfo:
    error_type: str
    error_message: str
    stack_trace: Optional[str]
    timestamp: datetime
    context: Dict[str, Any]
    severity: str  # 'low', 'medium', 'high', 'critical'
    category: str  # 'syntax', 'runtime', 'logic', 'timeout', 'dependency'
    
    def to_context_string(self) -> str:
        """转换为上下文字符串"""
```

### 重试上下文
```python
@dataclass
class RetryContext:
    errors: List[ErrorInfo]
    retry_count: int
    error_patterns: List[ErrorPattern]
    suggestions: List[str]
    
    def format_for_prompt(self) -> str:
        """格式化为prompt文本"""
```

### 错误模式
```python
@dataclass
class ErrorPattern:
    pattern_type: str
    frequency: int
    description: str
    suggested_fix: str
```

## 正确性属性

*属性是系统在所有有效执行中都应该保持为真的特征或行为——本质上是关于系统应该做什么的正式声明。属性是人类可读规范和机器可验证正确性保证之间的桥梁。*

### 属性1：错误信息完整性
*对于任何* 捕获的错误，系统应该保留足够的信息用于后续分析和重试
**验证需求：需求1.1, 1.2, 1.3, 1.4, 1.5**

### 属性2：上下文传递一致性
*对于任何* 重试操作，系统应该将相关的错误历史正确传递给模型
**验证需求：需求2.1, 2.2, 2.3**

### 属性3：错误分析准确性
*对于任何* 错误模式分析，系统应该提供准确和有用的错误分类和建议
**验证需求：需求3.1, 3.2, 3.3, 3.4, 3.5**

### 属性4：重试策略有效性
*对于任何* 重试决策，系统应该基于错误历史做出合理的策略调整
**验证需求：需求4.1, 4.2, 4.3, 4.4, 4.5**

## 错误处理

### 错误收集失败
- 当错误收集本身失败时，使用基础错误信息
- 记录收集失败的原因用于系统改进
- 确保不因错误收集失败而中断主流程

### 上下文传递失败
- 当上下文格式化失败时，提供简化版本
- 当prompt过长时，智能截断保留关键信息
- 提供降级策略确保重试能够继续

### 模式分析失败
- 当模式识别失败时，使用基础错误分类
- 提供手动模式定义的后备机制
- 记录分析失败案例用于算法改进

## 测试策略

### 单元测试
- 测试各种错误类型的捕获和格式化
- 测试错误上下文的生成和传递
- 测试prompt增强的正确性

### 属性测试
- 测试错误信息完整性在各种异常下的保持
- 测试上下文传递在不同重试场景下的一致性
- 测试错误分析在随机错误模式下的准确性

### 集成测试
- 测试完整的错误-重试-修正流程
- 测试与现有系统的集成
- 测试在实际使用场景下的效果