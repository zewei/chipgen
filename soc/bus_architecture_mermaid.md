# SOC Bus Architecture - Mermaid Diagram

## Interactive Bus Interconnect Diagram

```mermaid
graph TB
    subgraph "AXI Interconnect Matrix"
        AXI_XBAR[AXI Crossbar Switch]
        
        subgraph "AXI Masters"
            CPU_M[CPU Core<br/>Cortex-A35]
            DMA_M[DMA Controller]
            ETH_M[Ethernet MAC]
            USB_M[USB Controller]
            PCIE_M[PCIe Controller]
        end
        
        subgraph "AXI Slaves"
            DDR_S[DDR Controller<br/>64-bit AXI]
            SRAM_S[SRAM Controller<br/>32-bit AXI]
            QSPI_S[QSPI Flash Controller]
            AHB_BRIDGE[AHB-to-AXI Bridge]
        end
        
        CPU_M --> AXI_XBAR
        DMA_M --> AXI_XBAR
        ETH_M --> AXI_XBAR
        USB_M --> AXI_XBAR
        PCIE_M --> AXI_XBAR
        
        AXI_XBAR --> DDR_S
        AXI_XBAR --> SRAM_S
        AXI_XBAR --> QSPI_S
        AXI_XBAR --> AHB_BRIDGE
    end
    
    subgraph "AHB Interconnect Matrix"
        AHB_XBAR[AHB Crossbar Switch]
        
        subgraph "AHB Masters"
            AHB_BRIDGE_M[AXI-to-AHB Bridge]
            SDMMC_M[SDMMC Controller]
            MDMA_M[MDMA Controller]
        end
        
        subgraph "AHB Slaves"
            SRAM1[SRAM1<br/>128KB]
            SRAM2[SRAM2<br/>128KB]
            SRAM3[SRAM3<br/>64KB]
            SRAM4[SRAM4<br/>64KB]
            RETSRAM[Retention SRAM<br/>32KB]
            FLASH_CTRL[Flash Controller]
            APB_BRIDGE[APB-to-AHB Bridge]
        end
        
        AHB_BRIDGE_M --> AHB_XBAR
        SDMMC_M --> AHB_XBAR
        MDMA_M --> AHB_XBAR
        
        AHB_XBAR --> SRAM1
        AHB_XBAR --> SRAM2
        AHB_XBAR --> SRAM3
        AHB_XBAR --> SRAM4
        AHB_XBAR --> RETSRAM
        AHB_XBAR --> FLASH_CTRL
        AHB_XBAR --> APB_BRIDGE
    end
    
    subgraph "APB Interconnect Matrix"
        APB_XBAR[APB Crossbar Switch]
        
        subgraph "APB Masters"
            APB_BRIDGE_M[AHB-to-APB Bridge]
            DEBUG_M[Debug Interface]
        end
        
        subgraph "Timer APB Slaves"
            TIM1[TIM1<br/>Advanced Timer]
            TIM2[TIM2<br/>General Timer]
            TIM3[TIM3<br/>General Timer]
            TIM4[TIM4<br/>General Timer]
            TIM5[TIM5<br/>General Timer]
            LPTIM1[LPTIM1<br/>Low Power Timer]
            HRTIM[HRTIM<br/>High Resolution Timer]
        end
        
        subgraph "Communication APB Slaves"
            I2C1[I2C1<br/>400kHz]
            I2C2[I2C2<br/>400kHz]
            I2C3[I2C3<br/>400kHz]
            SPI1[SPI1<br/>50MHz]
            SPI2[SPI2<br/>50MHz]
            USART1[USART1<br/>115200bps]
            USART2[USART2<br/>115200bps]
            CAN1[CAN1<br/>1Mbps]
            CAN2[CAN2<br/>1Mbps]
        end
        
        subgraph "System APB Slaves"
            GPIOA[GPIOA<br/>Port A]
            GPIOB[GPIOB<br/>Port B]
            GPIOC[GPIOC<br/>Port C]
            GPIOD[GPIOD<br/>Port D]
            GPIOE[GPIOE<br/>Port E]
            GPIOF[GPIOF<br/>Port F]
            GPIOG[GPIOG<br/>Port G]
            GPIOH[GPIOH<br/>Port H]
            SYSCFG[SYSCFG<br/>System Config]
            PWR[PWR<br/>Power Control]
            RCC[RCC<br/>Reset & Clock]
            FLASH[FLASH<br/>Flash Config]
        end
        
        subgraph "Analog APB Slaves"
            ADC1[ADC1<br/>12-bit ADC]
            ADC2[ADC2<br/>12-bit ADC]
            DAC[DAC<br/>12-bit DAC]
            TEMP[TEMP<br/>Temperature Sensor]
        end
        
        APB_BRIDGE_M --> APB_XBAR
        DEBUG_M --> APB_XBAR
        
        APB_XBAR --> TIM1
        APB_XBAR --> TIM2
        APB_XBAR --> TIM3
        APB_XBAR --> TIM4
        APB_XBAR --> TIM5
        APB_XBAR --> LPTIM1
        APB_XBAR --> HRTIM
        
        APB_XBAR --> I2C1
        APB_XBAR --> I2C2
        APB_XBAR --> I2C3
        APB_XBAR --> SPI1
        APB_XBAR --> SPI2
        APB_XBAR --> USART1
        APB_XBAR --> USART2
        APB_XBAR --> CAN1
        APB_XBAR --> CAN2
        
        APB_XBAR --> GPIOA
        APB_XBAR --> GPIOB
        APB_XBAR --> GPIOC
        APB_XBAR --> GPIOD
        APB_XBAR --> GPIOE
        APB_XBAR --> GPIOF
        APB_XBAR --> GPIOG
        APB_XBAR --> GPIOH
        APB_XBAR --> SYSCFG
        APB_XBAR --> PWR
        APB_XBAR --> RCC
        APB_XBAR --> FLASH
        
        APB_XBAR --> ADC1
        APB_XBAR --> ADC2
        APB_XBAR --> DAC
        APB_XBAR --> TEMP
    end
    
    style AXI_XBAR fill:#f9f,stroke:#333,stroke-width:4px
    style AHB_XBAR fill:#9ff,stroke:#333,stroke-width:4px
    style APB_XBAR fill:#ff9,stroke:#333,stroke-width:4px
    style CPU_M fill:#f96,stroke:#333,stroke-width:2px
    style DDR_S fill:#9f9,stroke:#333,stroke-width:2px
    style SRAM1 fill:#99f,stroke:#333,stroke-width:2px
    style RCC fill:#f99,stroke:#333,stroke-width:2px
```

## Bus Performance Characteristics

| Bus Type | Data Width | Max Frequency | Throughput | Arbitration | Typical Use |
|----------|------------|---------------|------------|-------------|-------------|
| **AXI** | 64/128-bit | 1 GHz | 8-16 GB/s | Multi-layer | CPU, DMA, High-speed peripherals |
| **AHB** | 32/64-bit | 200 MHz | 800 MB/s-1.6 GB/s | Round-robin | SRAM, Flash, SDMMC |
| **APB** | 32-bit | 50 MHz | 200 MB/s | Single-master | Configuration registers |

## Bus Address Map (Simplified)

```
0x0000_0000 - 0x0FFF_FFFF : DDR Memory (256MB)
0x1000_0000 - 0x1FFF_FFFF : SRAM1-4, RETSRAM (512KB total)
0x2000_0000 - 0x2FFF_FFFF : Flash Memory (16MB)
0x4000_0000 - 0x4FFF_FFFF : APB Peripherals
  - 0x4000_0000 - 0x400F_FFFF : Timers
  - 0x4010_0000 - 0x401F_FFFF : Communication (I2C, SPI, USART, CAN)
  - 0x4020_0000 - 0x402F_FFFF : GPIO
  - 0x4030_0000 - 0x403F_FFFF : System (SYSCFG, PWR, RCC, FLASH)
  - 0x4040_0000 - 0x404F_FFFF : Analog (ADC, DAC, TEMP)
0x5000_0000 - 0x5FFF_FFFF : AHB Peripherals
0x6000_0000 - 0x6FFF_FFFF : QSPI Memory
0x7000_0000 - 0x7FFF_FFFF : System registers
```

## Key Architectural Features

### 1. **Hierarchical Bus Structure**
- **AXI**: High-performance system backbone
- **AHB**: Medium-performance peripheral bus  
- **APB**: Low-power configuration bus

### 2. **Bridge Architecture**
- AXI-to-AHB bridge enables seamless data transfer
- AHB-to-APB bridge provides register access
- Bridges handle protocol conversion and clock domain crossing

### 3. **Multi-Master Support**
- CPU, DMA, and peripherals can act as bus masters
- Arbitration ensures fair access and prevents conflicts
- Priority-based scheduling for real-time requirements

### 4. **Modular Design**
- Each peripheral type has dedicated bus segment
- Easy to add new masters/slaves without affecting existing connections
- Clean separation between high-speed and low-speed peripherals

### 5. **Performance Optimization**
- Separate buses for different performance classes
- Pipelined transactions on AXI for maximum throughput
- Burst transfers supported on AXI and AHB buses

This architecture provides a scalable, high-performance interconnect system suitable for complex SOC designs while maintaining low power consumption for battery-powered applications.