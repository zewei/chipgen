# Arch.yaml 参数合法性检查规则文档

## 概述
本文档描述了Arch.yaml文件中定义的参数合法性检查规则，这些规则用于验证SoC设计中各个模块参数的有效性和合理性。

## 规则分类

### 1. RCC参数验证规则 (rcc_parameter_rules)
专门用于验证时钟控制器的参数：

- **frequency_range_check**: RCC频率必须在1MHz到1GHz之间
- **pll_count_validation**: PLL数量必须在1到8之间  
- **clock_source_validation**: 时钟源数量必须在2到10之间

### 2. 时钟频率验证规则 (clock_frequency_rules)
验证不同类型模块的时钟频率参数：

- **cpu_frequency_range**: CPU时钟频率必须在1MHz到2GHz之间
- **peripheral_frequency_range**: 外设时钟频率建议在100kHz到500MHz之间
- **bus_frequency_range**: 总线时钟频率必须在1MHz到800MHz之间

### 3. 存储器参数验证规则 (memory_parameter_rules)
验证存储器相关参数：

- **flash_size_validation**: Flash大小必须在64KB到16MB之间
- **flash_sector_count**: Flash扇区数量建议在4到256之间
- **ddr_size_validation**: DDR大小必须在8MB到4GB之间

### 4. 外设参数验证规则 (peripheral_parameter_rules)
验证外设模块的参数：

- **gpio_pin_count**: GPIO端口引脚数量必须在1到32之间
- **timer_prescaler_range**: 定时器预分频器必须在0到65535之间
- **usart_baudrate_range**: USART波特率建议在300到3Mbps之间

### 5. 系统参数验证规则 (system_parameter_rules)
验证系统级参数：

- **power_domain_count**: 电源域数量必须在1到16之间
- **interrupt_priority_bits**: 中断优先级位宽必须在2到8之间
- **dma_channel_count**: DMA通道数量必须在1到32之间

### 6. 安全参数验证规则 (security_parameter_rules)
验证安全相关参数：

- **boot_mode_count**: 启动模式数量必须在2到16之间
- **exti_line_count**: 外部中断线数量必须在1到128之间
- **tamper_pin_count**: 防篡改引脚数量建议在0到8之间

### 7. 参数依赖关系验证规则 (parameter_dependency_rules)
验证参数之间的依赖关系：

- **pll_frequency_dependency**: PLL输出频率必须在VCO频率的1/128到1倍之间
- **clock_divider_dependency**: 时钟分频器输入频率必须是输出频率的整数倍
- **memory_timing_dependency**: 存储器时序参数必须满足CL ≤ tRCD ≤ tRP关系

### 8. 性能参数验证规则 (performance_parameter_rules)
验证性能相关参数：

- **max_frequency_limit**: 工作频率不能超过最大允许频率
- **power_consumption_limit**: 功耗超过建议最大值时发出警告
- **temperature_range_check**: 工作温度必须在-40°C到125°C之间

### 9. 兼容性检查规则 (compatibility_rules)
验证兼容性相关参数：

- **voltage_compatibility**: 工作电压必须是标准值(1.2V, 1.8V, 3.3V)
- **io_standard_compatibility**: 建议使用标准I/O电平标准

## 规则属性说明

每个验证规则包含以下属性：

- **rule**: 规则名称，必须唯一
- **description**: 规则描述，说明规则的作用
- **target**: 目标参数路径，支持通配符
- **condition**: 验证条件表达式
- **severity**: 严重程度，可选值为error或warning
- **message**: 验证失败时显示的错误消息
- **applies_to**: 可选，指定适用的实例列表

## 验证机制

1. **自动验证**: 在SoC设计处理过程中自动执行参数验证
2. **错误报告**: 验证失败时生成详细的错误报告，包括参数路径、当前值和期望范围
3. **严重程度分级**: 
   - Error: 严重错误，必须修复
   - Warning: 警告信息，建议修复

## 使用建议

1. **定期验证**: 在修改参数后重新运行验证
2. **渐进修复**: 优先修复Error级别的验证失败
3. **自定义规则**: 可根据具体项目需求添加自定义验证规则
4. **文档维护**: 保持验证规则与项目规范的一致性

## 扩展指南

添加新的验证规则时，请遵循以下原则：

1. **规则命名**: 使用有意义的规则名称，遵循snake_case命名规范
2. **条件表达式**: 确保条件表达式语法正确，考虑边界情况
3. **错误消息**: 提供清晰、具体的错误消息，帮助用户理解问题
4. **严重程度**: 根据问题的影响程度选择合适的严重程度
5. **测试验证**: 添加新规则后，使用测试用例验证规则的有效性