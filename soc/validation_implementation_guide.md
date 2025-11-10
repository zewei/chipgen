# 参数验证规则实现指南

## 概述
本文档描述了如何在QSoC框架中实现Arch.yaml中定义的参数合法性检查规则。

## 架构设计

### 1. 验证引擎架构
```
┌─────────────────────────────────────────────────────────────┐
│                    Validation Engine                       │
├─────────────────────────────────────────────────────────────┤
│  Rule Parser    │  Condition Evaluator  │  Report Generator │
├─────────────────────────────────────────────────────────────┤
│              Parameter Validator                           │
├─────────────────────────────────────────────────────────────┤
│              YAML Configuration Loader                     │
└─────────────────────────────────────────────────────────────┘
```

### 2. 核心组件

#### 2.1 规则解析器 (Rule Parser)
负责解析YAML格式的验证规则：

```cpp
class ValidationRule {
public:
    std::string rule_name;
    std::string description;
    std::string target_path;
    std::string condition;
    Severity severity;
    std::string message;
    std::vector<std::string> applies_to;
    
    bool matches(const std::string& instance_name) const;
    bool evaluate(const YAML::Node& parameter_value) const;
};
```

#### 2.2 条件评估器 (Condition Evaluator)
评估参数值是否满足条件表达式：

```cpp
class ConditionEvaluator {
public:
    bool evaluate(const std::string& condition, 
                  const YAML::Node& parameter_value,
                  const std::string& parameter_path) const;
    
private:
    std::unordered_map<std::string, std::function<bool(const YAML::Node&)>> 
        built_in_functions;
};
```

#### 2.3 参数验证器 (Parameter Validator)
执行具体的参数验证逻辑：

```cpp
class ParameterValidator {
public:
    ValidationResult validate(const YAML::Node& soc_config,
                              const std::vector<ValidationRule>& rules);
    
private:
    std::vector<ValidationError> errors;
    std::vector<ValidationWarning> warnings;
};
```

## 实现步骤

### 步骤1: 加载验证规则
```cpp
ValidationRuleLoader loader;
auto rules = loader.loadFromFile("Arch.yaml", "validation");
```

### 步骤2: 遍历参数
```cpp
void ParameterValidator::validateParameters(const YAML::Node& soc_config) {
    // 遍历所有实例
    for (const auto& instance : soc_config["instance"]) {
        std::string instance_name = instance.first.as<std::string>();
        
        // 检查实例参数
        if (instance.second["parameter"]) {
            validateInstanceParameters(instance_name, instance.second["parameter"]);
        }
    }
}
```

### 步骤3: 应用验证规则
```cpp
void ParameterValidator::validateInstanceParameters(
    const std::string& instance_name,
    const YAML::Node& parameters) {
    
    for (const auto& rule : validation_rules) {
        if (!rule.matches(instance_name)) continue;
        
        for (const auto& param : parameters) {
            std::string param_name = param.first.as<std::string>();
            std::string full_path = "instance." + instance_name + ".parameter." + param_name;
            
            if (rule.target_matches(full_path)) {
                applyRule(rule, param.second, full_path);
            }
        }
    }
}
```

### 步骤4: 条件评估
```cpp
bool ConditionEvaluator::evaluate(const std::string& condition,
                                  const YAML::Node& value,
                                  const std::string& path) const {
    // 解析条件表达式
    // 支持比较操作符: ==, !=, <, <=, >, >=
    // 支持逻辑操作符: &&, ||, !
    // 支持数学表达式: +, -, *, /, %
    // 支持函数调用: abs(), min(), max(), in_array()
    
    return parseAndEvaluate(condition, value, path);
}
```

## 支持的表达式语法

### 1. 基本比较
```yaml
condition: "value >= 1000000 && value <= 1000000000"
condition: "value == 400000000"
condition: "value != 0"
```

### 2. 字符串比较
```yaml
condition: "value in ['LVCMOS12', 'LVCMOS18', 'LVCMOS33']"
condition: "value == 'ENABLED' || value == 'DISABLED'"
```

### 3. 数学表达式
```yaml
condition: "value % 2 == 0"  # 偶数检查
condition: "abs(value - 1.2) <= 0.1"  # 近似值检查
```

### 4. 参数引用
```yaml
condition: "pll1_p_freq <= pll1_vco_freq"
condition: "input_freq >= output_freq && input_freq % output_freq == 0"
```

### 5. 复杂逻辑
```yaml
condition: "(value >= 1000000 && value <= 500000000) || value == 0"
condition: "!(value < 0 || value > 100)"
```

## 错误处理

### 1. 错误分类
- **语法错误**: 条件表达式语法不正确
- **类型错误**: 参数类型与预期不符
- **范围错误**: 参数值超出允许范围
- **依赖错误**: 参数依赖关系不满足

### 2. 错误报告格式
```cpp
struct ValidationError {
    std::string rule_name;
    std::string parameter_path;
    YAML::Node current_value;
    std::string condition;
    std::string message;
    Severity severity;
    std::vector<std::string> suggestions;
};
```

### 3. 错误恢复策略
- 遇到语法错误时跳过该规则，继续验证其他规则
- 记录所有错误和警告，不中断验证流程
- 提供详细的错误信息和修复建议

## 性能优化

### 1. 规则索引
```cpp
class RuleIndex {
private:
    std::unordered_map<std::string, std::vector<size_t>> instance_rules;
    std::unordered_map<std::string, std::vector<size_t>> parameter_rules;
    
public:
    void buildIndex(const std::vector<ValidationRule>& rules);
    std::vector<size_t> getApplicableRules(const std::string& path) const;
};
```

### 2. 缓存机制
- 缓存解析后的条件表达式
- 缓存频繁使用的参数值
- 缓存验证结果以避免重复计算

### 3. 并行验证
```cpp
void ParameterValidator::validateParallel(const YAML::Node& config) {
    std::vector<std::future<ValidationResult>> futures;
    
    for (const auto& instance : config["instance"]) {
        futures.push_back(std::async(std::launch::async,
            &ParameterValidator::validateInstance, this, instance));
    }
    
    for (auto& future : futures) {
        auto result = future.get();
        mergeResults(result);
    }
}
```

## 集成测试

### 1. 单元测试框架
```cpp
class ValidationTest {
public:
    void testBasicComparisons();
    void testStringOperations();
    void testMathematicalExpressions();
    void testParameterReferences();
    void testComplexLogic();
    void testErrorHandling();
};
```

### 2. 测试用例设计
- 正常值测试
- 边界值测试
- 异常值测试
- 依赖关系测试
- 性能压力测试

### 3. 测试覆盖率
- 规则覆盖率: 确保所有验证规则都被测试
- 条件覆盖率: 确保所有条件分支都被测试
- 参数覆盖率: 确保所有参数类型都被测试

## 部署指南

### 1. 编译集成
```cmake
# CMakeLists.txt
add_library(validation_engine
    ValidationRule.cpp
    ConditionEvaluator.cpp
    ParameterValidator.cpp
    ValidationReport.cpp
)

target_link_libraries(validation_engine
    yaml-cpp
    fmt::fmt
)
```

### 2. 配置选项
```yaml
validation:
  enable: true
  strict_mode: false
  max_errors: 100
  parallel_validation: true
  cache_enabled: true
  report_level: detailed
```

### 3. 运行时配置
```cpp
ValidationConfig config;
config.setStrictMode(false);
config.setMaxErrors(100);
config.setParallelValidation(true);
config.setCacheEnabled(true);

ParameterValidator validator(config);
```

## 维护建议

1. **规则更新**: 定期审查和更新验证规则
2. **性能监控**: 监控验证过程的性能指标
3. **错误分析**: 分析常见的验证失败模式
4. **用户反馈**: 收集用户对验证规则的反馈
5. **文档同步**: 保持实现与文档的同步更新