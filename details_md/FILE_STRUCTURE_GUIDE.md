# 📁 Intel® DeepInsight 项目文件结构说明

> **Intel® DeepInsight 智能零售决策系统** - 基于 Intel OpenVINO™ 与 DeepSeek R1 的全本地化自然语言数据分析平台

---

## 🏗️ 项目架构概览

```
Intel® DeepInsight/
├── 🚀 核心应用文件
├── 🧠 AI引擎模块
├── 📊 数据处理模块
├── 🔧 工具与优化
├── 🧪 测试文件
├── 📋 文档与报告
├── ⚙️ 配置与规格
└── 📁 数据目录
```

---

## 🚀 核心应用文件

### **app.py**
- **作用**: Streamlit主应用程序，系统的Web界面入口
- **功能**: 
  - 用户界面渲染和交互处理
  - 会话管理和消息历史
  - 配置管理和系统设置
  - 集成所有功能模块
- **重要性**: ⭐⭐⭐⭐⭐ (核心文件)

### **agent_core.py**
- **作用**: Text2SQL智能代理核心逻辑
- **功能**:
  - 自然语言到SQL的转换
  - 查询执行和结果处理
  - 多轮对话管理
  - 错误处理和重试机制
- **重要性**: ⭐⭐⭐⭐⭐ (核心文件)

---

## 🧠 AI引擎模块

### **rag_engine.py**
- **作用**: RAG (检索增强生成) 引擎
- **功能**:
  - 向量数据库管理
  - 语义检索和相似度匹配
  - 知识库构建和查询
  - OpenVINO优化的嵌入模型
- **重要性**: ⭐⭐⭐⭐⭐ (核心文件)

### **llm_client.py**
- **作用**: 大语言模型客户端
- **功能**:
  - DeepSeek API调用封装
  - 流式响应处理
  - 错误重试和异常处理
  - 模型配置管理
- **重要性**: ⭐⭐⭐⭐ (重要文件)

### **query_possibility_generator.py**
- **作用**: 查询可能性生成器
- **功能**:
  - 生成多种查询理解方式
  - 用户意图分析和歧义消解
  - 查询建议和自动补全
- **重要性**: ⭐⭐⭐ (功能增强)

---

## 📊 数据处理模块

### **table_selector.py**
- **作用**: 智能表选择器
- **功能**:
  - 基于查询内容自动选择相关数据表
  - 表关联关系分析
  - 语义相似度计算
- **重要性**: ⭐⭐⭐⭐ (重要文件)

### **data_filter.py**
- **作用**: 高级数据筛选器
- **功能**:
  - 交互式数据筛选界面
  - 条件保存和复用
  - 筛选结果导出
- **重要性**: ⭐⭐⭐ (功能增强)

### **visualization_engine.py**
- **作用**: 智能可视化引擎
- **功能**:
  - 自动图表类型识别
  - Plotly交互式图表生成
  - 图表样式和主题管理
- **重要性**: ⭐⭐⭐⭐ (重要文件)

### **export_manager.py**
- **作用**: 导出和分享管理器
- **功能**:
  - PDF报告生成
  - Excel/CSV数据导出
  - 会话分享功能
  - 图表导出处理
- **重要性**: ⭐⭐⭐ (功能增强)

---

## 🔧 工具与优化

### **intel_cpu_iris_optimizer.py**
- **作用**: Intel CPU + Iris Xe GPU优化器
- **功能**:
  - OpenVINO模型优化
  - 硬件加速配置
  - 性能基准测试
- **重要性**: ⭐⭐⭐⭐ (性能优化)

### **intel_optimization_integration.py**
- **作用**: Intel优化集成模块
- **功能**:
  - 优化功能统一接口
  - 硬件检测和配置
  - 性能监控集成
- **重要性**: ⭐⭐⭐ (性能优化)

### **performance_monitor.py**
- **作用**: 性能监控器
- **功能**:
  - 系统资源监控
  - 性能指标收集
  - 异常检测和告警
- **重要性**: ⭐⭐⭐ (监控工具)

### **anomaly_detector.py**
- **作用**: 智能异常检测器
- **功能**:
  - 统计异常检测
  - 业务异常识别
  - 趋势异常分析
- **重要性**: ⭐⭐⭐ (分析工具)

### **recommendation_engine.py**
- **作用**: 智能推荐引擎
- **功能**:
  - 后续问题推荐
  - 个性化建议生成
  - 用户行为分析
- **重要性**: ⭐⭐⭐ (用户体验)

---

## 🧪 测试文件

### 核心功能测试
- **test_system_integration.py**: 系统集成测试
- **test_final_integration.py**: 最终集成测试
- **test_intel_optimization.py**: Intel优化功能测试
- **test_three_improvements.py**: 三项改进功能测试

### 功能模块测试
- **test_enhanced_table_selection.py**: 增强表选择测试
- **test_table_selector_integration.py**: 表选择器集成测试
- **test_table_selection_persistence.py**: 表选择持久化测试
- **test_possibility_enumeration.py**: 可能性枚举测试
- **test_recommendation_*.py**: 推荐引擎相关测试
- **test_ai_recommendation_complete.py**: AI推荐完整测试

### 导出功能测试
- **test_enhanced_pdf_export.py**: 增强PDF导出测试
- **test_enhanced_export_with_charts.py**: 图表导出测试
- **test_chart_export_timeout.py**: 图表导出超时测试
- **test_pdf_visual_fix.py**: PDF视觉修复测试
- **test_json_serialization_fix.py**: JSON序列化修复测试

### UI和集成测试
- **test_app_chart_integration.py**: 应用图表集成测试
- **test_monitor_html.py**: 监控HTML测试
- **test_ux_features.py**: 用户体验功能测试

---

## 📋 文档与报告

### 功能完成报告
- **FINAL_UX_OPTIMIZATION_COMPLETE.md**: UX优化最终完成报告
- **FINAL_ENHANCED_EXPORT_COMPLETE.md**: 增强导出最终完成报告
- **FINAL_CRITICAL_FIXES_SUMMARY.md**: 关键修复最终总结
- **FINAL_FIXES_SUMMARY.md**: 修复总结最终版
- **FINAL_HTML_FIX_SUMMARY.md**: HTML修复最终总结

### 功能设计文档
- **ENHANCED_TEXT2SQL_DESIGN.md**: 增强Text2SQL设计文档
- **TEXT2SQL_POSSIBILITY_ENUMERATION_COMPLETE.md**: Text2SQL可能性枚举完成文档
- **NATURAL_LANGUAGE_POSSIBILITIES_COMPLETE.md**: 自然语言可能性完成文档
- **USER_CONFIGURABLE_POSSIBILITIES.md**: 用户可配置可能性文档

### 功能实现报告
- **INTELLIGENT_TABLE_SELECTION_COMPLETE.md**: 智能表选择完成报告
- **ENHANCED_TABLE_SELECTION_COMPLETE.md**: 增强表选择完成报告
- **ENHANCED_PDF_EXPORT_COMPLETE.md**: 增强PDF导出完成报告
- **INTEL_OPTIMIZATION_IMPLEMENTATION_REPORT.md**: Intel优化实现报告

### 修复和改进报告
- **THREE_IMPROVEMENTS_COMPLETE.md**: 三项改进完成报告
- **TWO_BUGS_FIXED_FINAL.md**: 两个Bug修复最终报告
- **AI_RECOMMENDATION_FIX_SUMMARY.md**: AI推荐修复总结
- **CHART_EXPORT_BUG_FIX.md**: 图表导出Bug修复
- **CHART_EXPORT_TIMEOUT_FIX.md**: 图表导出超时修复
- **JSON_SERIALIZATION_FIX.md**: JSON序列化修复
- **TABLE_SELECTION_PERSISTENCE_FIX.md**: 表选择持久化修复

### 配置和集成报告
- **RECOMMENDATION_CONFIG_ENHANCEMENT_SUMMARY.md**: 推荐配置增强总结
- **SEPARATE_API_CONFIG_FINAL_SUMMARY.md**: 独立API配置最终总结
- **TABLE_SELECTOR_INTEGRATION_FIX_SUMMARY.md**: 表选择器集成修复总结
- **UX_OPTIMIZATION_SUMMARY.md**: UX优化总结
- **LAYOUT_OPTIMIZATION_SUMMARY.md**: 布局优化总结

### 示例和演示
- **POSSIBILITY_ENUMERATION_EXAMPLE.md**: 可能性枚举示例
- **DETAILED_NATURAL_LANGUAGE_COMPLETE.md**: 详细自然语言完成文档

---

## ⚙️ 配置与规格

### .kiro/specs/ 目录
包含项目规格说明和任务定义：

#### Intel优化规格
- **.kiro/specs/intel-ux-optimization/**: UX优化规格
  - `requirements.md`: 需求文档
  - `design.md`: 设计文档
  - `tasks.md`: 任务列表

- **.kiro/specs/intel-competition-optimization/**: 竞赛优化规格
  - `requirements.md`: 需求文档
  - `design.md`: 设计文档
  - `tasks.md`: 任务列表

#### PDF视觉修复规格
- **.kiro/specs/pdf-visual-fix/**: PDF视觉修复规格
  - `requirements.md`: 需求文档
  - `design.md`: 设计文档
  - `tasks.md`: 任务列表

---

## 🛠️ 工具和脚本

### **demo_*.py** 文件
- **demo_enhanced_pdf_export.py**: PDF导出功能演示
- **demo_recommendation_config.py**: 推荐配置演示
- **demo_separate_api_config.py**: 独立API配置演示

### **utils.py**
- **作用**: 通用工具函数库
- **功能**: 常用工具函数和辅助方法

### **etl_pipeline.py**
- **作用**: ETL数据处理管道
- **功能**: 数据提取、转换和加载

### **optimize_model.py**
- **作用**: 模型优化脚本
- **功能**: OpenVINO模型转换和优化

### **benchmark_openvino.py**
- **作用**: OpenVINO性能基准测试
- **功能**: 性能测试和基准对比

---

## 📁 数据目录

### **data/** 目录
- **schema_northwind.json**: Northwind数据库Schema定义
- 其他数据文件和缓存文件

### **models/** 目录
- 存储AI模型文件和配置

### **cache/** 目录
- 系统缓存文件

### **assets/** 目录
- 静态资源文件

---

## 📄 配置文件

### **.env**
- **作用**: 环境变量配置文件
- **内容**: API密钥、数据库连接等敏感配置

### **requirements.txt**
- **作用**: Python依赖包列表
- **内容**: 项目所需的所有Python包及版本

### **license**
- **作用**: 项目许可证文件

---

## 🎥 演示材料

### **Intel® DeepInsight 智能零售决策系统.mp4**
- **作用**: 系统演示视频

### **Intel® DeepInsight 智能零售决策系统.pptx**
- **作用**: 系统介绍PPT

### **评分细则.md / 评分细则.docx**
- **作用**: 竞赛评分标准文档

---

## 🚀 快速启动指南

1. **安装依赖**: `pip install -r requirements.txt`
2. **配置环境**: 复制 `.env.example` 到 `.env` 并填写配置
3. **启动应用**: `streamlit run app.py`
4. **访问系统**: 浏览器打开 `http://localhost:8501`

---

## 📊 文件重要性等级

- ⭐⭐⭐⭐⭐ **核心文件**: app.py, agent_core.py, rag_engine.py
- ⭐⭐⭐⭐ **重要文件**: table_selector.py, visualization_engine.py, llm_client.py
- ⭐⭐⭐ **功能增强**: export_manager.py, recommendation_engine.py, 各种优化模块
- ⭐⭐ **工具脚本**: 测试文件、演示脚本、工具函数
- ⭐ **文档资料**: 各种MD文档、配置文件

---

*最后更新: 2025年1月*