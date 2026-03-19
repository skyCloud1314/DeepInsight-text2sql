# 🎯 增强版 Text2SQL 核心算法设计

## 📋 现有架构分析

### ✅ 您当前的优势
1. **OpenVINO加速的RAG检索** - 本地化向量搜索，性能优秀
2. **多轮重试机制** - 自动修复SQL错误
3. **意图分析** - 区分SQL查询和闲聊
4. **流式输出** - 良好的用户体验
5. **安全检查** - 只允许SELECT语句
6. **用户可配置的可能性探索** - 侧边栏中的`max_candidates`设置

### 🔄 可以改进的地方
1. **表结构选择过于简单** - 缺乏智能表关联分析
2. **上下文利用不充分** - 没有充分利用历史对话
3. **SQL优化策略单一** - 重试时缺乏结构化的优化策略
4. **缺乏查询复杂度评估** - 无法处理复杂的多表关联
5. **可能性探索功能未充分利用** - 现有的`max_candidates`只用于简单的歧义分析

## 🎯 增强版架构设计

### 核心思路：多阶段智能推理 + 用户配置驱动的可能性枚举

```
用户问题 → 问题理解 → 表结构智能选择 → 可能性枚举 → SQL构建 → 执行优化
    ↓         ↓           ↓              ↓           ↓         ↓
  意图分析   实体识别    关联分析      用户配置驱动   语法检查   性能优化
                                    (max_candidates)
```

## 🧠 详细设计方案

### 阶段1: 增强问题理解模块

#### 1.1 多维度实体识别
```python
class QueryAnalyzer:
    def analyze_query(self, query: str) -> QueryContext:
        return {
            "entities": self.extract_entities(query),      # 时间、地点、产品等
            "metrics": self.extract_metrics(query),        # 销售额、利润率等
            "operations": self.extract_operations(query),  # 排序、分组、筛选
            "time_range": self.extract_time_info(query),   # 时间范围
            "complexity": self.assess_complexity(query)    # 查询复杂度
        }
```

#### 1.2 业务领域知识增强
- **领域词典**: 建立业务术语到数据库字段的映射
- **同义词扩展**: "销售额"→["revenue", "sales", "turnover"]
- **时间表达式标准化**: "去年"→具体年份范围

### 阶段2: 智能表结构选择

#### 2.1 表关联图构建
```python
class SchemaGraph:
    def __init__(self, db_schema):
        self.tables = {}
        self.relationships = {}
        self.build_relationship_graph()
    
    def find_relevant_tables(self, entities: List[str]) -> List[TableInfo]:
        """基于实体和关联关系智能选择相关表"""
        candidate_tables = []
        for entity in entities:
            # 1. 直接匹配表名/字段名
            direct_matches = self.find_direct_matches(entity)
            # 2. 语义相似度匹配
            semantic_matches = self.find_semantic_matches(entity)
            # 3. 关联表推荐
            related_tables = self.find_related_tables(direct_matches)
            
            candidate_tables.extend([direct_matches, semantic_matches, related_tables])
        
        return self.rank_and_filter_tables(candidate_tables)
```

#### 2.2 动态Schema上下文构建
- **最小必要集**: 只包含查询相关的表和字段
- **关联路径**: 自动推荐表连接方式
- **字段描述增强**: 包含业务含义和示例值

### 阶段3: 查询计划生成

#### 3.1 查询复杂度分级
```python
class QueryPlanner:
    def classify_query_complexity(self, query_context: QueryContext) -> str:
        """
        SIMPLE: 单表查询，基础聚合
        MEDIUM: 多表JOIN，复杂WHERE条件
        COMPLEX: 子查询，窗口函数，复杂业务逻辑
        """
        score = 0
        if len(query_context.tables) > 1: score += 2
        if query_context.has_subquery: score += 3
        if query_context.has_window_function: score += 3
        if query_context.has_complex_aggregation: score += 2
        
        if score <= 2: return "SIMPLE"
        elif score <= 5: return "MEDIUM"
        else: return "COMPLEX"
```

#### 3.2 分层SQL生成策略
```python
def generate_sql_by_complexity(self, complexity: str, context: QueryContext) -> str:
    if complexity == "SIMPLE":
        return self.generate_simple_sql(context)
    elif complexity == "MEDIUM":
        return self.generate_medium_sql(context)
    else:
        return self.generate_complex_sql_with_cte(context)
```

### 阶段4: 用户配置驱动的可能性枚举

#### 4.1 基于max_candidates的智能可能性生成
```python
class ConfigurablePossibilityEnumerator:
    def __init__(self, max_candidates: int):
        self.max_candidates = max_candidates  # 从app.py侧边栏配置获取
        self.ambiguity_detector = AmbiguityDetector()
        self.possibility_ranker = PossibilityRanker()
    
    def enumerate_query_interpretations(self, query: str, context: EnhancedContext) -> List[QueryInterpretation]:
        """
        基于用户配置的max_candidates数量，生成查询的多种理解方式
        """
        if self.max_candidates == 1:
            # 用户只要1种可能性，直接返回最佳理解
            return [self.generate_best_interpretation(query, context)]
        
        # 1. 检测查询中的歧义点
        ambiguity_points = self.ambiguity_detector.detect_ambiguities(query, context)
        
        if not ambiguity_points:
            # 没有歧义，返回单一解释
            return [QueryInterpretation(
                query=query,
                interpretation="标准理解",
                confidence=1.0,
                ambiguity_resolutions={}
            )]
        
        # 2. 生成所有可能的理解方式
        all_interpretations = self.generate_all_interpretations(query, ambiguity_points, context)
        
        # 3. 根据用户配置的数量进行智能筛选
        ranked_interpretations = self.possibility_ranker.rank_interpretations(all_interpretations)
        
        # 4. 返回用户配置数量的可能性
        return ranked_interpretations[:self.max_candidates]
```

#### 4.2 歧义检测器
```python
class AmbiguityDetector:
    def __init__(self):
        # 内置的歧义模式，无需外部配置文件
        self.ambiguity_patterns = {
            "time_expressions": {
                "去年": ["2023年", "过去12个月", "上一财年"],
                "今年": ["2024年", "当前12个月", "本财年"],
                "最近": ["30天内", "90天内", "6个月内"],
                "上个月": ["上个自然月", "过去30天", "上个业务月"]
            },
            "business_metrics": {
                "销售额": ["总销售金额", "销售收入", "营业额"],
                "利润": ["净利润", "毛利润", "营业利润"],
                "利润率": ["净利润率", "毛利润率", "投资回报率"]
            },
            "aggregation_methods": {
                "最高": ["TOP 1", "MAX值", "排序第一"],
                "前几名": ["TOP N", "排名前列", "最高的几个"],
                "平均": ["算术平均", "加权平均", "中位数"]
            },
            "geographic_scope": {
                "地区": ["按州分组", "按城市分组", "按区域分组"],
                "全国": ["所有州", "美国境内", "包含所有地区"]
            }
        }
    
    def detect_ambiguities(self, query: str, context: EnhancedContext) -> Dict[str, List[str]]:
        """检测查询中的歧义点"""
        detected_ambiguities = {}
        
        for category, patterns in self.ambiguity_patterns.items():
            for ambiguous_term, possible_meanings in patterns.items():
                if ambiguous_term in query:
                    detected_ambiguities[ambiguous_term] = {
                        'category': category,
                        'possible_meanings': possible_meanings,
                        'context_relevance': self.assess_context_relevance(
                            possible_meanings, context
                        )
                    }
        
        return detected_ambiguities
```

#### 4.3 智能可能性排序
```python
class PossibilityRanker:
    def rank_interpretations(self, interpretations: List[QueryInterpretation]) -> List[QueryInterpretation]:
        """
        基于多个维度对可能性进行排序：
        1. 语义一致性 (40%)
        2. 数据可用性 (30%)
        3. 历史查询模式 (20%)
        4. 业务逻辑合理性 (10%)
        """
        for interpretation in interpretations:
            interpretation.confidence = self.calculate_comprehensive_score(interpretation)
        
        return sorted(interpretations, key=lambda x: x.confidence, reverse=True)
    
    def calculate_comprehensive_score(self, interpretation: QueryInterpretation) -> float:
        semantic_score = self.calculate_semantic_consistency(interpretation)
        data_score = self.calculate_data_availability(interpretation)
        history_score = self.calculate_history_match(interpretation)
        logic_score = self.calculate_business_logic_score(interpretation)
        
        return (
            semantic_score * 0.4 +
            data_score * 0.3 +
            history_score * 0.2 +
            logic_score * 0.1
        )
```

#### 4.4 与现有agent_core.py的集成
```python
# 在agent_core.py中的修改
class Text2SQLAgent:
    def __init__(self, ..., max_candidates: int = 3, ...):
        # ... 现有初始化代码 ...
        self.possibility_enumerator = ConfigurablePossibilityEnumerator(max_candidates)
    
    def generate_and_execute_stream(self, query: str, history_context: List[Dict]) -> Generator[Dict, None, None]:
        # ... 现有的RAG检索和意图分析 ...
        
        # 新增：可能性枚举阶段
        if self.max_candidates > 1:
            yield {"type": "step", "icon": "🤔", "msg": f"正在生成 {self.max_candidates} 种可能的理解方式...", "status": "running"}
            
            interpretations = self.possibility_enumerator.enumerate_query_interpretations(query, context)
            
            # 为每种理解生成SQL
            sql_candidates = []
            for i, interpretation in enumerate(interpretations):
                sql = self.generate_sql_for_interpretation(interpretation, context)
                sql_candidates.append({
                    'sql': sql,
                    'interpretation': interpretation,
                    'rank': i + 1
                })
            
            # 执行最佳候选SQL
            best_sql = sql_candidates[0]['sql']
            df, err = self.execute_sql(best_sql)
            
            if df is not None and not df.empty:
                # 成功，返回结果和其他可能性
                yield {"type": "result", "df": df, "sql": best_sql, "alternatives": sql_candidates[1:]}
                return
            else:
                # 最佳候选失败，尝试其他候选
                for candidate in sql_candidates[1:]:
                    df, err = self.execute_sql(candidate['sql'])
                    if df is not None and not df.empty:
                        yield {"type": "result", "df": df, "sql": candidate['sql'], "alternatives": []}
                        return
        
        # ... 继续现有的重试逻辑 ...
```

### 阶段5: 智能SQL优化与修复

#### 5.1 结构化错误修复
```python
class SQLOptimizer:
    def fix_sql_error(self, sql: str, error: str, attempt: int) -> str:
        error_type = self.classify_error(error)
        
        if error_type == "SYNTAX_ERROR":
            return self.fix_syntax_error(sql, error)
        elif error_type == "COLUMN_NOT_FOUND":
            return self.fix_column_reference(sql, error)
        elif error_type == "TABLE_NOT_FOUND":
            return self.fix_table_reference(sql, error)
        elif error_type == "EMPTY_RESULT":
            return self.relax_conditions(sql, attempt)
        else:
            return self.generic_fix(sql, error)
```

#### 5.2 查询结果优化
- **空结果智能处理**: 逐步放宽条件
- **性能优化**: 添加合适的索引提示
- **结果验证**: 检查结果的合理性

### 阶段6: 上下文记忆与学习

#### 6.1 对话上下文管理
```python
class ConversationContext:
    def __init__(self):
        self.query_history = []
        self.successful_patterns = {}
        self.user_preferences = {}
    
    def update_context(self, query: str, sql: str, success: bool):
        """学习成功的查询模式"""
        if success:
            pattern = self.extract_pattern(query, sql)
            self.successful_patterns[pattern] = self.successful_patterns.get(pattern, 0) + 1
```

#### 6.2 个性化优化
- **用户偏好学习**: 记住用户常用的时间范围、指标等
- **查询模式复用**: 相似问题复用成功的SQL模式
- **错误模式避免**: 记住常见错误，提前规避

## 🛠️ 实现建议

### 核心模块重构

#### 1. 增强RAG引擎
```python
class EnhancedRAG(IntelRAG):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.schema_graph = SchemaGraph(self.db_schema)
        self.query_analyzer = QueryAnalyzer()
        self.domain_knowledge = DomainKnowledge()
    
    def intelligent_retrieve(self, query: str) -> EnhancedContext:
        # 1. 问题分析
        query_context = self.query_analyzer.analyze_query(query)
        
        # 2. 智能表选择
        relevant_tables = self.schema_graph.find_relevant_tables(
            query_context.entities
        )
        
        # 3. 构建增强上下文
        enhanced_context = self.build_enhanced_context(
            query_context, relevant_tables
        )
        
        return enhanced_context
```

#### 2. 升级SQL生成器
```python
class EnhancedSQLGenerator:
    def __init__(self):
        self.query_planner = QueryPlanner()
        self.sql_optimizer = SQLOptimizer()
        self.conversation_context = ConversationContext()
        self.possibility_enumerator = PossibilityEnumerator("config/ambiguity_config.json")
    
    def generate_sql_with_possibilities(self, query: str, context: EnhancedContext) -> SQLGenerationResult:
        # 1. 枚举所有可能的理解方式
        possibilities = self.possibility_enumerator.enumerate_possibilities(query, context)
        
        # 2. 为最高置信度的可能性生成SQL
        best_possibility = possibilities[0]
        primary_sql = self.generate_sql_for_possibility(best_possibility, context)
        
        # 3. 为其他可能性生成备选SQL
        alternative_sqls = []
        for possibility in possibilities[1:4]:  # 最多3个备选
            alt_sql = self.generate_sql_for_possibility(possibility, context)
            alternative_sqls.append({
                "sql": alt_sql,
                "description": possibility.get_natural_description(),
                "confidence": possibility.confidence_score
            })
        
        return SQLGenerationResult(
            primary_sql=primary_sql,
            alternatives=alternative_sqls,
            ambiguous_terms=self.possibility_enumerator.get_ambiguous_terms(query)
        )
```

#### 3. 用户交互增强
```python
class UserInteractionManager:
    def present_alternatives_to_user(self, result: SQLGenerationResult) -> UserChoice:
        """
        在UI中展示可能性选项，让用户确认或选择
        """
        # 1. 执行主要SQL
        primary_result = self.execute_sql(result.primary_sql)
        
        # 2. 如果有歧义术语，展示选择界面
        if result.ambiguous_terms:
            user_choice = self.show_ambiguity_resolution_ui(
                primary_result=primary_result,
                alternatives=result.alternatives,
                ambiguous_terms=result.ambiguous_terms
            )
            return user_choice
        
        return UserChoice(selected_sql=result.primary_sql, confirmed=True)
    
    def show_ambiguity_resolution_ui(self, primary_result, alternatives, ambiguous_terms):
        """
        UI展示逻辑：
        1. 显示主要查询结果
        2. 展示"其他可能的理解方式"卡片
        3. 用户可以点击切换到其他理解方式
        4. 记录用户选择，用于后续学习
        """
        pass
```

### 性能优化策略

#### 1. 缓存机制
- **查询模式缓存**: 相似查询直接复用SQL
- **表结构缓存**: 避免重复扫描数据库
- **向量缓存**: 常用查询的向量表示

#### 2. 并行处理
- **异步RAG检索**: 并行进行向量搜索和表结构分析
- **多候选生成**: 同时生成多个SQL候选，选择最优

## 🎯 预期效果

### 准确性提升
- **表选择准确率**: 从60%提升到85%+
- **SQL生成成功率**: 从70%提升到90%+
- **复杂查询处理**: 支持多表JOIN、子查询等

### 用户体验改善
- **响应速度**: 智能缓存减少重复计算
- **错误恢复**: 结构化修复策略，更快收敛
- **个性化**: 学习用户偏好，提供定制化服务

### 系统鲁棒性
- **错误处理**: 分类处理不同类型错误
- **边界情况**: 优雅处理空结果、超时等
- **可扩展性**: 模块化设计，易于添加新功能

## 🚀 实施路线图

### Phase 1: 核心增强 + 可能性枚举 (2-3周)
1. 实现QueryAnalyzer - 实体识别和查询分析
2. 构建SchemaGraph - 智能表选择
3. **实现PossibilityEnumerator - 可能性枚举核心模块**
4. **创建配置表管理系统 - 支持歧义术语配置**
5. 升级RAG检索逻辑

### Phase 2: SQL优化 + 用户交互 (1-2周)
1. 实现QueryPlanner - 复杂度分级
2. 增强SQL生成策略
3. **实现UserInteractionManager - 用户选择界面**
4. **集成可能性展示到前端UI**
5. 结构化错误修复

### Phase 3: 上下文学习 + 智能排序 (1周)
1. 对话上下文管理
2. 查询模式学习
3. **实现PossibilityRanker - 智能置信度评分**
4. **用户选择反馈学习机制**
5. 个性化优化

### Phase 4: 性能优化 + 系统集成 (1周)
1. 缓存机制实现
2. 并行处理优化
3. **可能性生成性能优化**
4. **配置表热更新机制**
5. 系统性能调优

这个设计保持了您现有架构的优势，同时在智能性、准确性和用户体验方面都有显著提升。您觉得这个方向如何？我们可以从哪个模块开始实现？