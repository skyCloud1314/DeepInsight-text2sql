# 🎯 用户配置驱动的可能性枚举系统

## 📋 与现有系统的集成

### 现有配置项利用
您在app.py侧边栏中已经有了这个配置：
```python
max_candidates = st.slider("可能性探索 (条)", 1, 5, st.session_state.config.get("max_candidates", 3))
```

我们将充分利用这个用户配置，让系统根据用户设置的数量来生成和展示不同的查询理解方式。

## 🛠️ 具体实现方案

### 1. 增强agent_core.py中的可能性处理

```python
class Text2SQLAgent:
    def __init__(self, ..., max_candidates: int = 3, ...):
        # ... 现有代码 ...
        self.max_candidates = max_candidates
        self.possibility_generator = QueryPossibilityGenerator()
    
    def generate_and_execute_stream(self, query: str, history_context: List[Dict]) -> Generator[Dict, None, None]:
        # ... 现有的RAG检索和意图分析代码 ...
        
        # ---------------------------------------------------------
        # 新增阶段：基于用户配置的可能性枚举
        # ---------------------------------------------------------
        if self.max_candidates > 1:
            yield {"type": "step", "icon": "🎯", "msg": f"正在生成 {self.max_candidates} 种可能的查询理解...", "status": "running"}
            yield {"type": "thought_start"}
            
            # 生成多种可能的理解方式
            possibilities = self.possibility_generator.generate_possibilities(
                query=query, 
                context=context, 
                max_count=self.max_candidates
            )
            
            # 流式输出思考过程
            for possibility in possibilities:
                thought_content = f"理解方式 {possibility.rank}: {possibility.description}\n"
                thought_content += f"置信度: {possibility.confidence:.1%}\n"
                thought_content += f"关键解释: {possibility.key_interpretations}\n\n"
                yield {"type": "thought_chunk", "content": thought_content}
            
            yield {"type": "step", "icon": "✅", "msg": f"已生成 {len(possibilities)} 种理解方式", "status": "complete"}
            
            # 按置信度顺序尝试执行
            for i, possibility in enumerate(possibilities):
                try:
                    sql = self.generate_sql_for_possibility(possibility, context)
                    df, err = self.execute_sql(sql)
                    
                    if df is not None and not df.empty:
                        # 成功执行，返回结果和备选方案
                        alternatives = [p for p in possibilities if p != possibility]
                        yield {
                            "type": "result", 
                            "df": df, 
                            "sql": sql,
                            "selected_possibility": possibility,
                            "alternatives": alternatives
                        }
                        return
                    elif i == 0:
                        # 最佳理解也失败了，记录错误继续尝试
                        yield {"type": "error_log", "content": f"最佳理解执行失败: {err}"}
                        
                except Exception as e:
                    if i == 0:
                        yield {"type": "error_log", "content": f"最佳理解生成失败: {str(e)}"}
                    continue
        
        # 如果所有可能性都失败，回退到原有的重试机制
        # ... 继续现有的重试逻辑 ...
```

### 2. 查询可能性生成器

```python
class QueryPossibilityGenerator:
    def __init__(self):
        # 内置的歧义识别规则，无需外部配置文件
        self.ambiguity_rules = {
            "time_ambiguity": {
                "去年": {
                    "interpretations": [
                        {"desc": "2023年全年", "sql_condition": "strftime('%Y', order_date) = '2023'", "confidence": 0.9},
                        {"desc": "过去12个月", "sql_condition": "order_date >= date('now', '-12 months')", "confidence": 0.7},
                        {"desc": "上一财年", "sql_condition": "order_date >= '2023-01-01' AND order_date <= '2023-12-31'", "confidence": 0.6}
                    ]
                },
                "今年": {
                    "interpretations": [
                        {"desc": "2024年至今", "sql_condition": "strftime('%Y', order_date) = '2024'", "confidence": 0.9},
                        {"desc": "最近12个月", "sql_condition": "order_date >= date('now', '-12 months')", "confidence": 0.6},
                        {"desc": "本财年至今", "sql_condition": "order_date >= '2024-01-01'", "confidence": 0.7}
                    ]
                },
                "最近": {
                    "interpretations": [
                        {"desc": "最近30天", "sql_condition": "order_date >= date('now', '-30 days')", "confidence": 0.8},
                        {"desc": "最近90天", "sql_condition": "order_date >= date('now', '-90 days')", "confidence": 0.7},
                        {"desc": "最近6个月", "sql_condition": "order_date >= date('now', '-6 months')", "confidence": 0.6}
                    ]
                }
            },
            "metric_ambiguity": {
                "销售额": {
                    "interpretations": [
                        {"desc": "总销售金额", "sql_expression": "SUM(sales)", "confidence": 0.9},
                        {"desc": "销售收入", "sql_expression": "SUM(sales * quantity)", "confidence": 0.7},
                        {"desc": "营业额", "sql_expression": "SUM(unit_price * quantity)", "confidence": 0.6}
                    ]
                },
                "利润": {
                    "interpretations": [
                        {"desc": "净利润", "sql_expression": "SUM(profit)", "confidence": 0.9},
                        {"desc": "毛利润", "sql_expression": "SUM(sales - cost)", "confidence": 0.7},
                        {"desc": "营业利润", "sql_expression": "SUM((unit_price - cost_price) * quantity)", "confidence": 0.6}
                    ]
                }
            },
            "ranking_ambiguity": {
                "最高": {
                    "interpretations": [
                        {"desc": "单个最高值", "sql_pattern": "ORDER BY {metric} DESC LIMIT 1", "confidence": 0.9},
                        {"desc": "最高的记录", "sql_pattern": "WHERE {metric} = (SELECT MAX({metric}) FROM table)", "confidence": 0.7}
                    ]
                },
                "前几名": {
                    "interpretations": [
                        {"desc": "前5名", "sql_pattern": "ORDER BY {metric} DESC LIMIT 5", "confidence": 0.8},
                        {"desc": "前10名", "sql_pattern": "ORDER BY {metric} DESC LIMIT 10", "confidence": 0.6},
                        {"desc": "前3名", "sql_pattern": "ORDER BY {metric} DESC LIMIT 3", "confidence": 0.7}
                    ]
                }
            }
        }
    
    def generate_possibilities(self, query: str, context: str, max_count: int) -> List[QueryPossibility]:
        """根据用户配置的max_count生成可能性"""
        
        # 1. 检测查询中的歧义点
        detected_ambiguities = self.detect_ambiguities_in_query(query)
        
        if not detected_ambiguities:
            # 没有歧义，返回单一标准理解
            return [QueryPossibility(
                rank=1,
                description="标准理解",
                confidence=1.0,
                key_interpretations={},
                ambiguity_resolutions={}
            )]
        
        # 2. 生成所有可能的组合
        all_combinations = self.generate_interpretation_combinations(detected_ambiguities)
        
        # 3. 计算每种组合的综合置信度
        for combo in all_combinations:
            combo.confidence = self.calculate_combined_confidence(combo)
        
        # 4. 排序并返回用户指定数量的可能性
        sorted_combinations = sorted(all_combinations, key=lambda x: x.confidence, reverse=True)
        
        # 5. 为每个可能性分配排名
        for i, combo in enumerate(sorted_combinations[:max_count]):
            combo.rank = i + 1
        
        return sorted_combinations[:max_count]
    
    def detect_ambiguities_in_query(self, query: str) -> Dict[str, List[Dict]]:
        """检测查询中的歧义术语"""
        detected = {}
        
        for category, rules in self.ambiguity_rules.items():
            for ambiguous_term, rule_config in rules.items():
                if ambiguous_term in query:
                    detected[ambiguous_term] = {
                        'category': category,
                        'interpretations': rule_config['interpretations']
                    }
        
        return detected
    
    def generate_interpretation_combinations(self, ambiguities: Dict) -> List[QueryPossibility]:
        """生成所有可能的解释组合"""
        if not ambiguities:
            return []
        
        combinations = []
        ambiguous_terms = list(ambiguities.keys())
        
        # 使用递归生成所有组合
        def generate_recursive(term_index: int, current_combination: Dict):
            if term_index >= len(ambiguous_terms):
                # 生成一个完整的可能性
                possibility = QueryPossibility(
                    rank=0,  # 稍后分配
                    description=self.create_description(current_combination),
                    confidence=0.0,  # 稍后计算
                    key_interpretations=current_combination.copy(),
                    ambiguity_resolutions=current_combination.copy()
                )
                combinations.append(possibility)
                return
            
            term = ambiguous_terms[term_index]
            interpretations = ambiguities[term]['interpretations']
            
            for interpretation in interpretations:
                current_combination[term] = interpretation
                generate_recursive(term_index + 1, current_combination)
                del current_combination[term]
        
        generate_recursive(0, {})
        return combinations
    
    def create_description(self, combination: Dict[str, Dict]) -> str:
        """为解释组合创建人类可读的描述"""
        desc_parts = []
        for term, interpretation in combination.items():
            desc_parts.append(f"{term}→{interpretation['desc']}")
        return " | ".join(desc_parts) if desc_parts else "标准理解"
    
    def calculate_combined_confidence(self, possibility: QueryPossibility) -> float:
        """计算组合解释的综合置信度"""
        if not possibility.key_interpretations:
            return 1.0
        
        # 使用几何平均数来计算综合置信度
        confidences = [interp['confidence'] for interp in possibility.key_interpretations.values()]
        if confidences:
            import math
            geometric_mean = math.pow(math.prod(confidences), 1.0 / len(confidences))
            return geometric_mean
        return 0.5

@dataclass
class QueryPossibility:
    rank: int
    description: str
    confidence: float
    key_interpretations: Dict[str, Dict]
    ambiguity_resolutions: Dict[str, Dict]
    generated_sql: Optional[str] = None
```

### 3. 前端UI集成

在app.py的历史消息渲染中添加可能性展示：

```python
# 在历史消息渲染部分添加
if "alternatives" in msg and msg["alternatives"]:
    st.markdown("##### 🤔 其他可能的理解方式")
    
    with st.expander(f"查看其他 {len(msg['alternatives'])} 种理解方式", expanded=False):
        for i, alt in enumerate(msg["alternatives"]):
            col1, col2 = st.columns([4, 1])
            
            with col1:
                st.write(f"**理解方式 {alt.rank}**: {alt.description}")
                st.write(f"置信度: {alt.confidence:.1%}")
                if alt.key_interpretations:
                    interpretations_text = " | ".join([
                        f"{term}: {interp['desc']}" 
                        for term, interp in alt.key_interpretations.items()
                    ])
                    st.caption(f"关键解释: {interpretations_text}")
            
            with col2:
                if st.button(f"选择此理解", key=f"select_alt_{msg_index}_{i}"):
                    # 重新执行这种理解方式
                    st.session_state.prompt_trigger = f"[重新理解] {msg['query']} [按照: {alt.description}]"
                    st.rerun()
```

## 🎯 用户体验流程

### 场景1: 用户设置max_candidates=1
- 系统直接生成最佳理解的SQL
- 不显示其他可能性选项
- 执行速度最快

### 场景2: 用户设置max_candidates=3
- 系统生成3种可能的理解方式
- 执行置信度最高的理解
- 在UI中展示其他2种备选理解
- 用户可以点击切换到其他理解

### 场景3: 用户设置max_candidates=5
- 系统生成5种可能的理解方式
- 按置信度排序，依次尝试执行
- 展示所有备选理解方式
- 提供最全面的选择

## 🚀 实际应用示例

**用户查询**: "去年科技产品的销售额最高的5个城市"
**用户配置**: max_candidates = 3

**系统生成的3种理解**:
1. **理解方式1** (置信度: 87%): 2023年全年 | 总销售金额 | 前5名城市
2. **理解方式2** (置信度: 72%): 过去12个月 | 总销售金额 | 前5名城市  
3. **理解方式3** (置信度: 68%): 2023年全年 | 销售收入 | 前5名城市

**UI展示**:
```
🎯 查询结果 (理解方式1: 2023年全年 | 总销售金额 | 前5名城市)
[显示数据表格]

🤔 其他可能的理解方式 ▼
理解方式2: 过去12个月 | 总销售金额 | 前5名城市 (置信度: 72%)    [选择此理解]
理解方式3: 2023年全年 | 销售收入 | 前5名城市 (置信度: 68%)      [选择此理解]
```

这样的设计完全基于您现有的配置系统，让用户可以根据需要调整可能性探索的深度！