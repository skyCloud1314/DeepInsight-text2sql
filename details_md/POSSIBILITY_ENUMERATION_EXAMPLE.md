# 🎯 可能性枚举模块 - 具体实现示例

## 📋 核心思路

当用户输入一个可能有歧义的查询时，系统会：
1. **识别歧义点** - 基于配置表找出可能有多种理解的术语
2. **枚举所有可能性** - 生成所有合理的理解组合
3. **智能排序** - 根据置信度排序，选择最可能的执行
4. **展示备选项** - 在UI中展示其他可能性，供用户选择

## 🛠️ 具体实现

### 1. 配置表结构

```json
{
  "time_expressions": {
    "去年": {
      "possibilities": ["2023", "last_year", "previous_year"],
      "sql_mappings": [
        "strftime('%Y', order_date) = '2023'",
        "order_date >= date('now', '-1 year', 'start of year') AND order_date < date('now', 'start of year')",
        "order_date >= date('2023-01-01') AND order_date <= date('2023-12-31')"
      ]
    },
    "最近": {
      "possibilities": ["last_30_days", "last_90_days", "last_6_months"],
      "sql_mappings": [
        "order_date >= date('now', '-30 days')",
        "order_date >= date('now', '-90 days')",
        "order_date >= date('now', '-6 months')"
      ]
    }
  },
  "business_metrics": {
    "销售额": {
      "possibilities": ["sales", "revenue", "total_sales"],
      "sql_mappings": [
        "SUM(sales)",
        "SUM(sales * quantity)",
        "SUM(unit_price * quantity)"
      ]
    },
    "利润": {
      "possibilities": ["profit", "net_profit", "gross_profit"],
      "sql_mappings": [
        "SUM(profit)",
        "SUM(sales - cost)",
        "SUM((unit_price - cost_price) * quantity)"
      ]
    }
  },
  "geographic_regions": {
    "加州": {
      "possibilities": ["California", "CA", "West_Coast"],
      "sql_mappings": [
        "state = 'California'",
        "state = 'CA'",
        "region = 'West' AND state IN ('California', 'Oregon', 'Washington')"
      ]
    }
  },
  "product_categories": {
    "科技产品": {
      "possibilities": ["Technology", "Electronics", "Tech_Hardware"],
      "sql_mappings": [
        "category = 'Technology'",
        "category IN ('Technology', 'Electronics')",
        "sub_category LIKE '%Tech%' OR category = 'Technology'"
      ]
    }
  }
}
```

### 2. 可能性枚举器实现

```python
class PossibilityEnumerator:
    def __init__(self, config_path: str):
        with open(config_path, 'r', encoding='utf-8') as f:
            self.config = json.load(f)
        self.max_possibilities = 5  # 最多生成5种可能性
    
    def enumerate_query_possibilities(self, query: str, schema_context: str) -> List[QueryPossibility]:
        """
        主入口：枚举查询的所有可能理解方式
        """
        # 1. 识别查询中的歧义术语
        ambiguous_terms = self.identify_ambiguous_terms(query)
        
        if not ambiguous_terms:
            # 没有歧义，返回单一可能性
            return [QueryPossibility(
                query=query,
                interpretations={},
                confidence_score=1.0,
                description="标准理解"
            )]
        
        # 2. 生成所有可能的组合
        possibilities = self.generate_possibility_combinations(query, ambiguous_terms)
        
        # 3. 计算置信度并排序
        for possibility in possibilities:
            possibility.confidence_score = self.calculate_confidence(possibility, schema_context)
        
        possibilities.sort(key=lambda x: x.confidence_score, reverse=True)
        
        return possibilities[:self.max_possibilities]
    
    def identify_ambiguous_terms(self, query: str) -> Dict[str, List[str]]:
        """识别查询中的歧义术语"""
        ambiguous_terms = {}
        
        for category, terms in self.config.items():
            for term, config in terms.items():
                if term in query:
                    ambiguous_terms[term] = {
                        'category': category,
                        'possibilities': config['possibilities'],
                        'sql_mappings': config['sql_mappings']
                    }
        
        return ambiguous_terms
    
    def generate_possibility_combinations(self, query: str, ambiguous_terms: Dict) -> List[QueryPossibility]:
        """生成所有可能的理解组合"""
        possibilities = []
        
        # 获取所有歧义术语的可能性
        term_names = list(ambiguous_terms.keys())
        term_possibilities = [ambiguous_terms[term]['possibilities'] for term in term_names]
        
        # 生成笛卡尔积（所有组合）
        from itertools import product
        for combination in product(*term_possibilities):
            interpretations = {}
            description_parts = []
            
            for i, term in enumerate(term_names):
                selected_possibility = combination[i]
                interpretations[term] = {
                    'selected': selected_possibility,
                    'category': ambiguous_terms[term]['category'],
                    'sql_mapping': ambiguous_terms[term]['sql_mappings'][
                        ambiguous_terms[term]['possibilities'].index(selected_possibility)
                    ]
                }
                description_parts.append(f"{term}→{selected_possibility}")
            
            possibility = QueryPossibility(
                query=query,
                interpretations=interpretations,
                confidence_score=0.0,  # 稍后计算
                description=" | ".join(description_parts)
            )
            possibilities.append(possibility)
        
        return possibilities
    
    def calculate_confidence(self, possibility: QueryPossibility, schema_context: str) -> float:
        """计算可能性的置信度分数"""
        score = 0.0
        
        # 1. 基础分数
        base_score = 0.5
        
        # 2. 语义相似度分数 (使用简单的关键词匹配)
        semantic_score = self.calculate_semantic_similarity(possibility, schema_context)
        
        # 3. 历史偏好分数 (如果有用户历史数据)
        history_score = self.calculate_history_preference(possibility)
        
        # 4. 数据可用性分数 (检查相关字段是否存在)
        availability_score = self.calculate_data_availability(possibility, schema_context)
        
        # 综合评分
        total_score = (
            base_score * 0.2 +
            semantic_score * 0.4 +
            history_score * 0.2 +
            availability_score * 0.2
        )
        
        return min(1.0, max(0.0, total_score))
```

### 3. 查询可能性数据结构

```python
@dataclass
class QueryPossibility:
    query: str                          # 原始查询
    interpretations: Dict[str, Any]     # 歧义术语的解释
    confidence_score: float             # 置信度分数
    description: str                    # 人类可读的描述
    generated_sql: Optional[str] = None # 生成的SQL
    execution_result: Optional[Any] = None # 执行结果
    
    def get_natural_description(self) -> str:
        """生成自然语言描述"""
        if not self.interpretations:
            return "标准理解"
        
        desc_parts = []
        for term, interpretation in self.interpretations.items():
            category = interpretation['category']
            selected = interpretation['selected']
            
            if category == 'time_expressions':
                desc_parts.append(f"时间范围: {selected}")
            elif category == 'business_metrics':
                desc_parts.append(f"指标: {selected}")
            elif category == 'geographic_regions':
                desc_parts.append(f"地区: {selected}")
            elif category == 'product_categories':
                desc_parts.append(f"产品类别: {selected}")
        
        return " | ".join(desc_parts)
    
    def apply_interpretations_to_sql_template(self, sql_template: str) -> str:
        """将解释应用到SQL模板中"""
        sql = sql_template
        for term, interpretation in self.interpretations.items():
            sql_mapping = interpretation['sql_mapping']
            # 替换SQL模板中的占位符
            sql = sql.replace(f"{{term}}", sql_mapping)
        return sql
```

### 4. 用户界面集成

```python
class PossibilityUI:
    def display_query_possibilities(self, possibilities: List[QueryPossibility]) -> QueryPossibility:
        """
        在Streamlit界面中展示可能性选项
        """
        # 执行最高置信度的查询
        best_possibility = possibilities[0]
        
        # 显示主要结果
        st.markdown("##### 🎯 查询结果 (最可能的理解)")
        st.info(f"理解方式: {best_possibility.get_natural_description()}")
        
        if best_possibility.execution_result is not None:
            st.dataframe(best_possibility.execution_result)
        
        # 如果有其他可能性，显示选择界面
        if len(possibilities) > 1:
            st.markdown("##### 🤔 其他可能的理解方式")
            
            with st.expander("点击查看其他理解方式", expanded=False):
                for i, possibility in enumerate(possibilities[1:], 1):
                    col1, col2 = st.columns([3, 1])
                    
                    with col1:
                        st.write(f"**理解方式 {i+1}**: {possibility.get_natural_description()}")
                        st.write(f"置信度: {possibility.confidence_score:.2%}")
                    
                    with col2:
                        if st.button(f"选择此理解", key=f"select_possibility_{i}"):
                            return possibility
        
        return best_possibility
```

## 🎯 实际使用示例

### 示例1: 时间歧义
**用户输入**: "去年科技产品的销售额是多少？"

**系统识别的歧义**:
- "去年" → [2023年, 过去12个月, 上一财年]
- "科技产品" → [Technology类别, Electronics类别, 包含Tech的所有产品]
- "销售额" → [sales字段, sales*quantity, unit_price*quantity]

**生成的可能性**:
1. **最可能** (置信度: 85%): 2023年 | Technology类别 | sales字段
2. **可能性2** (置信度: 72%): 过去12个月 | Technology类别 | sales字段  
3. **可能性3** (置信度: 68%): 2023年 | Electronics类别 | sales字段

### 示例2: 地理歧义
**用户输入**: "加州最近的家具销售情况"

**系统识别的歧义**:
- "加州" → [California州, CA缩写, 西海岸地区]
- "最近" → [30天, 90天, 6个月]
- "销售情况" → [销售额, 销售量, 利润]

**UI展示**:
```
🎯 查询结果 (最可能的理解)
理解方式: 地区: California | 时间范围: last_30_days | 指标: sales

[显示数据表格]

🤔 其他可能的理解方式
理解方式 2: 地区: West_Coast | 时间范围: last_90_days | 指标: sales     [选择此理解]
理解方式 3: 地区: California | 时间范围: last_6_months | 指标: profit   [选择此理解]
```

## 🚀 优势与价值

### 1. 提升查询准确性
- **减少误解**: 用户可以看到系统的理解方式，及时纠正
- **覆盖更多场景**: 处理自然语言的固有歧义性

### 2. 改善用户体验  
- **透明化**: 用户了解系统的推理过程
- **可控性**: 用户可以选择符合意图的理解方式
- **学习性**: 系统记住用户偏好，逐步提升准确性

### 3. 业务价值
- **降低错误成本**: 避免基于错误理解做出业务决策
- **提升信任度**: 用户对系统结果更有信心
- **扩展适用性**: 支持更复杂、更模糊的业务查询

这个可能性枚举模块将大大增强您的Text2SQL系统的智能性和实用性！