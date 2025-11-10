# SOC Bus Interconnect Architecture Diagram

## Overview
This diagram illustrates the bus interconnect architecture extracted from `bus.yaml`, showing the hierarchical connection between different bus types and their connected modules.

```
                                SOC Bus Interconnect Architecture
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│                                    AXI Interconnect                                    │
│  ┌─────────────────────────────────────────────────────────────────────────────────┐  │
│  │                            High-Speed AXI Bus Matrix                                │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │  │
│  │  │ DDR Master  │  │ ETH Master  │  │ USB Master  │  │ PCIE Master │          │  │
│  │  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘          │  │
│  │         │               │               │               │                 │  │
│  │  ┌──────▼──────┐  ┌──────▼──────┐  ┌──────▼──────┐  ┌──────▼──────┐          │  │
│  │  │ DDR Slave   │  │ ETH Slave   │  │ USB Slave   │  │ PCIE Slave  │          │  │
│  │  │ (ddr_ctrl)  │  │   (eth)     │  │   (usb)     │  │   (pcie)    │          │  │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘          │  │
│  └─────────────────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────────────────┘
                                           │
                                           ▼
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│                                   AHB Interconnect                                     │
│  ┌─────────────────────────────────────────────────────────────────────────────────┐  │
│  │                             AHB Bus Matrix                                      │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │  │
│  │  │SRAM Master  │  │SDMMC Master  │  │             │  │             │          │  │
│  │  └──────┬──────┘  └──────┬──────┘  │             │  │             │          │  │
│  │         │               │        │             │  │             │          │  │
│  │  ┌──────▼──────┐  ┌──────▼──────┐  │   Other     │  │   Other     │          │  │
│  │  │SRAM1 Slave  │  │SDMMC1 Slave │  │   Masters   │  │   Slaves    │          │  │
│  │  │SRAM2 Slave  │  │SDMMC2 Slave │  │             │  │             │          │  │
│  │  │SRAM3 Slave  │  └─────────────┘  │             │  │             │          │  │
│  │  │SRAM4 Slave  │                    └─────────────┘  └─────────────┘          │  │
│  │  │RETSRAM Slave│                                                              │  │
│  │  │Flash Slave   │                                                              │  │
│  │  └─────────────┘                                                              │  │
│  └─────────────────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────────────────┘
                                           │
                                           ▼
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│                                   APB Interconnect                                     │
│  ┌─────────────────────────────────────────────────────────────────────────────────┐  │
│  │                             APB Bus Matrix                                      │  │
│  │                                                                                 │  │
│  │  ┌──────────────┬──────────────┬──────────────┬──────────────┬──────────────┐  │  │
│  │  │ Timer Slaves │  DMA Slaves  │ GPIO Slaves  │Comm Slaves   │System Slaves │  │  │
│  │  │              │              │              │              │              │  │  │
│  │  │tim1_cfg      │dma1_cfg      │gpioa_cfg     │i2c1_cfg      │syscfg        │  │  │
│  │  │tim2_cfg      │dma2_cfg      │gpiob_cfg     │i2c2_cfg      │flash         │  │  │
│  │  │tim3_cfg      │bdma_cfg      │gpioc_cfg     │i2c3_cfg      │pwr           │  │  │
│  │  │tim4_cfg      │mdma_cfg      │gpiod_cfg     │i2c4_cfg      │dbgmcu        │  │  │
│  │  │tim5_cfg      │              │gpioe_cfg     │i2c5_cfg      │temp          │  │  │
│  │  │tim6_cfg      │              │gpiof_cfg     │              │              │  │  │
│  │  │tim7_cfg      │              │gpiog_cfg     │spi1_cfg      │              │  │  │
│  │  │tim8_cfg      │              │gpioh_cfg     │spi2_cfg      │              │  │  │
│  │  │tim9_cfg      │              │              │spi3_cfg      │              │  │  │
│  │  │tim10_cfg     │              │              │spi4_cfg      │              │  │  │
│  │  │tim11_cfg     │              │              │spi5_cfg      │              │  │  │
│  │  │tim12_cfg     │              │              │              │              │  │  │
│  │  │tim13_cfg     │              │              │usart1_cfg    │              │  │  │
│  │  │tim14_cfg     │              │              │usart2_cfg    │              │  │  │
│  │  │tim15_cfg     │              │              │usart3_cfg    │              │  │  │
│  │  │tim16_cfg     │              │              │uart4_cfg     │              │  │  │
│  │  │tim17_cfg     │              │              │uart5_cfg     │              │  │  │
│  │  │              │              │              │usart6_cfg    │              │  │  │
│  │  │lptim1_cfg    │              │              │uart7_cfg     │              │  │  │
│  │  │lptim2_cfg    │              │              │uart8_cfg     │              │  │  │
│  │  │              │              │              │              │              │  │  │
│  │  │hrtim_cfg     │              │              │can1_cfg      │              │  │  │
│  │  │              │              │              │can2_cfg      │              │  │  │
│  │  └──────────────┴──────────────┴──────────────┴──────────────┴──────────────┘  │  │
│  └─────────────────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────────────────┘
```

## Bus Hierarchy Summary

### 1. AXI Bus (High Performance)
- **DDR AXI**: Connects DDR controller to AXI interconnect
- **Ethernet AXI**: High-speed Ethernet MAC interface
- **USB AXI**: USB 3.0/2.0 controller interface  
- **PCIe AXI**: PCI Express controller interface

### 2. AHB Bus (Medium Performance) 
- **SRAM AHB**: Connects all SRAM modules (SRAM1-4, RETSRAM)
- **SDMMC AHB**: SD/MMC card interfaces
- **Flash AHB**: Flash memory controller

### 3. APB Bus (Low Performance)
- **Timer APB**: 17 general timers + 2 low-power timers + 1 high-res timer
- **DMA APB**: 4 DMA controllers (DMA1/2, BDMA, MDMA)
- **GPIO APB**: 8 GPIO ports (A-H)
- **Communication APB**: 
  - I2C: 5 interfaces
  - SPI: 5 interfaces  
  - USART/UART: 8 interfaces
  - CAN: 2 interfaces
- **System APB**: Configuration and control modules

## Key Features
- **Hierarchical Bus Architecture**: AXI → AHB → APB
- **Performance-based Segregation**: High-speed peripherals on AXI, medium-speed on AHB, low-speed on APB
- **Modular Design**: Each peripheral type has dedicated bus segment
- **Scalable Structure**: Easy to add new masters/slaves to each bus matrix

## Bus Characteristics
| Bus Type | Width | Frequency | Purpose |
|----------|--------|-----------|----------|
| AXI | 64/128-bit | Up to 1GHz | High-speed data transfer |
| AHB | 32/64-bit | Up to 200MHz | Medium-speed peripherals |
| APB | 32-bit | Up to 50MHz | Low-speed configuration |

This architecture provides optimal performance balancing between different peripheral types while maintaining clean separation of concerns and ease of system integration.