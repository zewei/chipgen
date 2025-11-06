
# STM32MP25x Architecture Overview

## Top-level
- **External clocks**: HSE (16-48 MHz), LSE (32.768 kHz)
- **External reset**: NRST (active low, open-drain, 20kΩ pull-up)
- **Boot selection**: BOOT[3:0] pins with OTP override capability

## IP Selection and Counts

### Processor Cores
- **Cortex-A35**: 
  - Single-core (STM32MP251x) or Dual-core (STM32MP253x/255x/257x)
  - Frequency: 1.2 GHz (STM32MP25xA) or 1.5 GHz (STM32MP25xD)
  - 32KB I-cache + 32KB D-cache per core, 512KB unified L2 cache
  - Armv8-A architecture with AArch32 and AArch64 support
- **Cortex-M33**: 
  - Single core, 400 MHz
  - 16KB I-cache + 16KB D-cache, FPU, MPU, TrustZone
- **Cortex-M0+**: 
  - Single core, up to 200 MHz (16 MHz in backup mode)
  - Located in SmartRun domain

### Graphics and AI (Optional)
- **GPU**: VeriSilicon GC8000UL
  - Frequency: 800 MHz (STM32MP25xA) or 900 MHz (STM32MP25xD)
  - APIs: OpenGL ES 3.1, Vulkan 1.3, OpenCL 3.0, OpenVX 1.3
  - Performance: 133/150 Miriangle/s, 800/900 Mpixel/s
- **NPU**: VeriSilicon (shared with GPU)
  - Frequency: 800 MHz (STM32MP25xA) or 900 MHz (STM32MP25xD)
  - Performance: 1.2/1.35 TOPS (8-bit integer)
  - APIs: TensorFlowLite, ONNX, Linux NN

### Video Processing (Optional)
- **Video Encoder/Decoder**: Up to 600 MHz
  - H264/VP8 up to 1080p60 (performance shared between VENC and VDEC)
  - JPEG up to 500 Mpixel/s
  - 128KB shared video RAM (VDERAM)

### Communication Interfaces
- **I2C**: 8x (1 Mbit/s, SMBus/PMBus support)
- **I3C**: 4x (MIPI I3C v1.1 compliant)
- **USART**: 4x + UART: 5x (12.5 Mbit/s)
  - USART1/2/3/6 + UART4/5/7/8/9
- **SPI**: 8x (50 Mbit/s, 3x with full-duplex I2S)
- **SAI**: 4x (serial audio interfaces, stereo audio: I2S, PDM, SPDIF Tx)
- **Ethernet**: 
  - ETH1: Gigabit GMAC with one PHY interface (optional)
  - ETH2: Gigabit GMAC with external PHY interface
  - ETHSW: 3-port Gigabit Ethernet switch (optional, in STM32MP257x)
- **USB**:
  - USBH: USB 2.0 Host with embedded 480 Mbps PHY
  - USB3DR: USB 2.0/3.0 Dual Role with embedded 480 Mbps + 5 Gbps PHY
  - UCPD1: USB Type-C Power Delivery controller
- **PCIe**: 1 lane Gen2, embedded 5 Gbps PHY (shared with USB 3.0)
- **FDCAN**: Up to 3x (CAN FD, 1x supports TTCAN)

### Storage Interfaces
- **SDMMC**: 3x (SD/eMMC/SDIO)
  - SDMMC1: 4/8-bit, usually for SD-Card
  - SDMMC2: 4/8-bit, usually for e-MMC
  - SDMMC3: 4-bit
- **OCTOSPI**: 2x (serial NOR/NAND, HyperFlash)
- **FMC**: Flexible memory controller
  - Parallel interface: 8/16-bit, up to 4×64MB
  - NAND Flash: 8/16-bit, SLC, BCH4/8 ECC

### Display and Camera
- **LCD-TFT**: 
  - Up to FHD (1920×1080) at 60 fps
  - 24-bit RGB888, 3 layers including secure layer
  - YUV support, 90° output rotation
- **MIPI DSI** (optional): 
  - 4 lanes, up to 2.5 Gbit/s each
  - Up to QXGA (2048×1536) at 60 fps
- **LVDS** (optional):
  - Single-link or dual-link, 4 data lanes per link, up to 1.1 Gbit/s per lane
  - Up to QXGA (2048×1536) at 60 fps with dual-link
- **Camera**:
  - CSI-2: 2 lanes, up to 2.5 Gbit/s each, up to 5MP@30fps
  - Parallel: 8-16 bit, up to 120 MHz, up to 1MP@15fps
  - DCMIPP with Lite-ISP (demosaicing, downscaling, cropping)

### Analog and Security
- **ADC**: 3x 12-bit, up to 5 Msps each, up to 23 channels total
- **Security**:
  - TrustZone for Cortex-A35 and Cortex-M33
  - HASH: SHA-1, SHA-224, SHA-256, SHA3, HMAC
  - PKA: ECDSA verification
  - RNG: True random number generator, FIPS 140-2 compliant
  - Active tamper detection, environmental monitors
- **Digital filters**:
  - MDF1: 8 input channels with 8 filters
  - ADF1: 1 input channel with 1 filter and sound activity detection


## Clocks

### Clock Sources

#### External Oscillators
- **HSE**: 16-48 MHz crystal oscillator (required for DDR and USB)
- **LSE**: 32.768 kHz crystal oscillator (RTC)

#### Internal Oscillators
- **HSI**: ~64 MHz RC oscillator (±1% accuracy)
- **MSI**: ~4/16 MHz multi-speed RC oscillator
- **LSI**: ~32 kHz low-speed RC oscillator

### PLL Configuration

#### Main PLLs
- **PLL1**: Dedicated for Cortex-A35 (up to 1.5 GHz)
- **PLL2**: Dedicated for GPU/NPU (up to 900 MHz) 
- **PLL3**: Dedicated for DDR controller (up to 1200 MHz)
- **PLL4/5/6/7/8**: For system and peripheral clocks
  - Integer or fractional mode
  - Spread-spectrum modulation for EMI reduction
  - Dynamic frequency scaling support

### Clock Distribution Tree

#### High-Frequency Domains
- **Cortex-A35 domain**: 
  - Source: Dedicated PLL1
  - Frequency: Up to 1.5 GHz
  - Clock Gating: Per-core, per-cache level

- **GPU/NPU domain**:
  - Source: Dedicated PLL2  
  - Frequency: Up to 900 MHz
  - Dynamic Voltage Frequency Scaling (DVFS)

- **DDR domain**:
  - Source: Dedicated PLL3
  - Frequency: Up to 1200 MHz
  - Self-refresh capability for low-power modes

#### System Interconnect Domains
- **STNoC Interconnect**: 128/64-bit multi-layer AXI
  - Frequencies: 600/300/400/200 MHz domains
  - QoS: 16 levels, 3 traffic classes per direction

- **Cortex-M33 domain**:
  - Frequency: Up to 400 MHz
  - Bus Matrix: 32-bit multi-AHB at 400 MHz and 200 MHz

- **Peripheral Domains**:
  - APB1/2/3/4: 200 MHz
  - AHB buses: 200-400 MHz depending on domain

#### Low-Power Domains
- **SmartRun domain**:
  - Cortex-M0+: Up to 200 MHz (16 MHz in backup mode)
  - Autonomous operation in low-power modes

- **Backup domain**:
  - LSI: ~32 kHz or LSE: 32.768 kHz
  - Always-on for RTC, tamper, backup registers

### Clock Links and Relationships

```
HSE (16-48MHz) ──┬─→ PLL1 ──→ Cortex-A35 (1.5GHz)
                 ├─→ PLL2 ──→ GPU/NPU (900MHz)
                 ├─→ PLL3 ──→ DDR (1200MHz)
                 ├─→ PLL4/5/6/7/8 ──→ System Clocks
                 └─→ Direct ──→ Peripheral Clocks

HSI (64MHz) ────┬─→ Backup for HSE failure
                └─→ Low-power operation

LSE (32.768kHz) ─→ RTC, Tamper, Backup domain
LSI (32kHz) ─────→ Independent Watchdog, Backup domain
```

## Resets

### Reset Sources Hierarchy

#### Primary Reset Sources
- **Power-on reset (POR)**: Monitors VDD and VDDA18AON
- **Power-down reset (PDR)**: Monitors VDD and VDDA18AON  
- **Brownout reset (BOR)**: Optional 2.7V threshold (disabled when VDD = 1.8V)
- **External NRST pin**: Active low, open-drain, 20kΩ internal pull-up
- **Independent watchdogs**: IWDG1/2/3/4/5 (LSI clocked)
- **System window watchdogs**: WWDG1/2 (APB clocked)
- **Software reset**: Via RCC control registers
- **HSE clock failure**: Automatic fallback to HSI
- **RETRAM CRC or ECC errors**: Data integrity failures

#### Domain-Specific Reset Supervisors
- **POR_VDDCORE/PDR_VDDCORE**: VDDCORE domain monitoring
- **POR_VDDCPU/PDR_VDDCPU**: VDDCPU (Cortex-A35) domain monitoring  
- **POR_VSW**: VSW domain monitoring
- **Peripheral voltage monitoring**: VDDIO1/2/3/4, VDD33USB, VDD33UCPD, VDDA18ADC

### Reset Domains and Scope

#### System Reset (Full Chip Reset)
- **Sources**: 
  - Application reset sources
  - Low-voltage detection on VDDCORE
  - Low-voltage detection on VDDCPU
- **Scope**: All domains except backup domain
- **Effect**: Complete system reinitialization

#### Application Reset
- **Sources**: 
  - NRST pin assertion
  - Low-voltage detection on VDD
  - Independent watchdog timeout
  - Software reset command
  - HSE failure
  - RETRAM CRC/ECC error
- **Scope**: Main application domains

#### Local Resets
- Individual peripheral resets via RCC
- Domain-specific reset controls
- TrustZone-aware reset partitioning

### Reset Links and Relationships

```
NRST ──┬─→ System Reset ──┬─→ Cortex-A35 Domain
       │                  ├─→ Cortex-M33 Domain  
       │                  ├─→ Peripheral Domains
       │                  └─→ Interconnect
       │
       ├─→ Application Reset ─→ Selected Domains
       │
       └─→ Local Resets ──┬─→ Individual Peripherals
                          ├─→ DMA Controllers
                          └─→ Memory Controllers

Watchdogs ──┬─→ IWDG1-5 ─→ System/Application Reset
            └─→ WWDG1-2 ─→ Application Reset

Power Monitors ──┬─→ POR/PDR ─→ System Reset
                 ├─→ BOR ──→ System Reset
                 └─→ PVD ──→ Interrupt/Wake-up only
```

### Reset Distribution by Domain

#### Cortex-A35 Subsystem Reset
- **Controlled by**: POR_VDDCPU/PDR_VDDCPU
- **Synchronization**: To A35 clock domain
- **Scope**: A35 cores, L1/L2 caches, SCU, GIC, generic timers

#### Cortex-M33 Subsystem Reset  
- **Controlled by**: System reset + local reset controller
- **Synchronization**: To M33 clock domain
- **Scope**: M33 core, NVIC, MPU, FPU, SysTick

#### Cortex-M0+ SmartRun Domain Reset
- **Controlled by**: Separate reset controller
- **Synchronization**: To M0+ clock domain
- **Scope**: M0+ core, LPSRAM, LPDMA, low-power peripherals

#### Peripheral Domain Resets
- AXI/AHB/APB peripheral resets
- Individual reset bits in RCC registers
- Clock-domain synchronized de-assertion
- TrustZone access control

#### Backup Domain Reset
- **Independent reset control**
- **Maintained during VBAT operation**
- **Scope**: RTC, tamper, backup registers, backup SRAM, LSE, LSI

### Reset Characteristics
- **Assertion**: Asynchronous for immediate response
- **De-assertion**: Synchronized to respective clock domains
- **Polarity**: Active low for most signals
- **TrustZone**: Secure and non-secure reset controls
- **Debug**: Freeze capability in debug mode for most resets

## Bus Interconnect Architecture

### STNoC Multi-Frequency Network (128/64-bit AXI)

#### Main System Interconnect
- **Topology**: Multi-layer AXI interconnect
- **Data Width**: 128-bit and 64-bit paths
- **Operating Frequencies**: 600/300/400/200 MHz domains
- **QoS Support**: Up to 16 QoS levels, 3 traffic classes per direction
- **Firewalling**: CID-based with poisoning output signaling

#### STNoC Masters
- **Cortex-A35 Subsystem (CA35SS)**
  - Dual Cortex-A35 cores with SCU (Snoop Control Unit)
  - 512KB L2 cache controller
  - Generic Interrupt Controller (GIC)
- **GPU** (optional) - VeriSilicon GC8000UL
- **NPU** (optional) - VeriSilicon neural processor
- **HPDMA1** - 16 channels, AXI master + AHB master
- **HPDMA2** - 16 channels, AXI master + AHB master  
- **HPDMA3** - 16 channels, AXI master + AHB master
- **Video Encoder (VENC)** (optional)
- **Video Decoder (VDEC)** (optional)
- **LCD-TFT Display Controller (LTDC)**
- **MIPI DSI** (optional)
- **LVDS Display Interface** (optional)
- **Digital Camera Interface with Pixel Processing (DCMIPP)**
- **Camera Serial Interface (CSI)**
- **Ethernet MAC 1 (ETH1)**
- **Ethernet MAC 2 (ETH2)**
- **USB 3.0 Dual Role (USB3DR)**
- **PCI Express (PCIE)**

#### STNoC Slaves
- **DDR Controller (DDRCTRL)** - 2×128-bit AXI4 ports
  - Supports DDR3L, DDR4, LPDDR4
  - Up to 4GB capacity
- **SYSRAM** - 256KB internal SRAM
- **VDERAM** - 128KB video RAM (shared with VENC/VDEC)
- **APB Bridges** - For peripheral access
  - APB1 Bridge (basic peripherals)
  - APB2 Bridge (advanced peripherals)
  - APB3 Bridge (system peripherals)
  - APB4 Bridge (security peripherals)
- **AHB Bridges** - For MCU domain connectivity
  - MCU MLAHB 400MHz bridge
  - MCU MLAHB 200MHz bridge

### MCU Multi-Layer AHB (32-bit)

#### 400 MHz MLAHB (Cortex-M33 Fast Domain)

**Masters:**
- **Cortex-M33 Core**
  - I-Fast interface (instruction fetch)
  - I-Slow interface (instruction access)
  - S-Bus interface (system access)
  - D-Fetch interface (data access)
- **HPDMA1/2/3** - Through OCTOSPI memory access path
- **MPU Matrix** - From Cortex-A35 domain

**Slaves:**
- **SRAM1** - 128KB with hardware erase on tamper
- **SRAM2** - 128KB
- **RETRAM** - 128KB with ECC/CRC protection
- **OCTOSPI1/2 Register Interfaces**
- **Connection to MPU Matrix** - For Cortex-A35 access
- **Connection to 200 MHz MLAHB** - Bridge to peripheral domain

#### 200 MHz MLAHB (Peripheral Domain)

**Masters:**
- **400 MHz MLAHB Bridge** - From Cortex-M33 fast domain
- **HPDMA1/2/3** - Direct access to peripherals

**Slaves:**
- **APB1 Bridge** - Timers, I2C, basic peripherals
- **APB2 Bridge** - Advanced peripherals  
- **APB3 Bridge** - System peripherals
- **APB4 Bridge** - Security peripherals
- **AHB3 Bridge** - Additional peripherals
- **AHB4 Bridge** - System functions
- **AHB5 Bridge** - Connectivity peripherals
- **AHB6 Bridge** - External interfaces
- **Connection to SmartRun AHB Matrix** - Bridge to low-power domain
- **Connection to MPU Matrix** - For Cortex-A35 access

### SmartRun Multi-Layer AHB (32-bit)

#### 200/16 MHz MLAHB (Low-Power Domain)

**Masters:**
- **Cortex-M0+ Processor**
- **LPDMA1 Controller** - 4 channels, AHB master
- **MCU 200 MHz MLAHB Bridge** - From peripheral domain

**Slaves:**
- **LPSRAM1** - 8KB with retention
- **LPSRAM2** - 8KB with retention  
- **LPSRAM3** - 16KB
- **APB SmartRun Bridge** - Low-power peripherals
- **PWR** - Power management
- **IPCC2** - Inter-processor communication controller
- **HSEM** - Hardware semaphore
- **EXTI2** - Extended interrupts
- **ADF1** - Audio digital filter
- **GPIOZ** - Special GPIOs
- **Connection to AHB3 RIFSC Submatrix** - Resource isolation

### Peripheral Bus Hierarchy

#### APB1 (200 MHz) - Basic Peripherals
- **Timers**:
  - TIM2, TIM3, TIM4, TIM5 (32-bit general purpose)
  - TIM6, TIM7 (16-bit basic)
  - TIM10, TIM11, TIM13, TIM14 (16-bit general purpose)
  - TIM12 (16-bit general purpose)
  - LPTIM1, LPTIM2 (low-power)
- **Communication**:
  - I2C1, I2C2, I2C3, I2C4, I2C5, I2C6, I2C7, I2C8
  - USART2, USART3
  - UART4, UART5
  - SPI2, SPI3 (with I2S2, I2S3)
  - SPDIFRX
  - I3C1, I3C2, I3C3

#### APB2 (200 MHz) - Advanced Peripherals
- **Timers**:
  - TIM1, TIM8, TIM20 (16-bit advanced motor control)
  - TIM15, TIM16, TIM17 (16-bit general purpose)
- **Communication**:
  - USART1, USART6
  - UART7, UART8, UART9
  - SPI1 (with I2S1)
  - SPI4, SPI5, SPI6, SPI7, SPI8
  - SAI1, SAI2, SAI3, SAI4
  - FDCAN1, FDCAN2, FDCAN3
- **Digital Filters**:
  - MDF1 (multi-function digital filter)
  - ADF1 (audio digital filter)

#### APB3 (200 MHz) - System Peripherals
- **System Control**:
  - SYSCFG (system configuration)
  - RCC (reset and clock control)
  - PWR (power control)
  - TAMP (tamper and backup)
  - RTC (real-time clock)
- **System Services**:
  - CRC (cyclic redundancy check)
  - HSEM (hardware semaphore)
  - IPCC1 (inter-processor communication)

#### APB4 (200 MHz) - Security Peripherals
- **Cryptography**:
  - HASH processor
  - PKA (public key accelerator)
  - RNG (random number generator)
- **Security Management**:
  - BSEC (boot and security OTP control)
  - RIF (resource isolation framework)

### AHB Peripheral Mapping

#### AHB1/2/3 (200-400 MHz) - Memory and High-Speed Peripherals
- **GPIO Ports**: GPIOA through GPIOK
- **ADC**: ADC1, ADC2, ADC3
- **SDMMC**: SDMMC1, SDMMC2, SDMMC3
- **DCMI**: Digital camera interface
- **CRYP**: Cryptographic processor
- **HASH**: Hash processor
- **RNG**: Random number generator

#### AHB4 (200 MHz) - System Functions
- **CRC**: Cyclic redundancy check
- **DMA1/2**: DMA controllers
- **FLASH**: Flash memory interface
- **SRAM1/2**: Static RAM controllers

### DMA Subsystem

#### High-Performance DMA (HPDMA)
- **Instances**: 3 (HPDMA1, HPDMA2, HPDMA3)
- **Channels**: 16 channels each (48 total)
- **Bus Interfaces**: AXI master + AHB master
- **Transfer Types**:
  - Peripheral-to-memory
  - Memory-to-peripheral  
  - Memory-to-memory
  - Peripheral-to-peripheral
- **Features**:
  - Autonomous operation in Sleep/Stop modes
  - Per-channel event and interrupt generation
  - Linked-list support
  - TrustZone and privilege support
  - Channel isolation
  - FIFO buffering (per channel)

#### Low-Power DMA (LPDMA)
- **Instance**: 1 (LPDMA1)
- **Channels**: 4 channels
- **Bus Interface**: AHB master
- **Transfer Types**:
  - Peripheral-to-memory
  - Memory-to-peripheral
  - Memory-to-memory
  - Peripheral-to-peripheral
- **Features**:
  - Autonomous operation in low-power modes
  - Linked-list support
  - TrustZone and privilege support
  - Channel isolation

### Resource Isolation Framework (RIF)

#### Hardware Isolation
- **Compartments**: 8 hardware execution compartments
- **Access Control**: 
  - Privileged/unprivileged access control
  - Secure/non-secure state control
  - Compartment-based resource allocation
- **Scope**:
  - Memory regions (embedded buffers, external memory)
  - Peripherals (all configurable peripherals)
  - System resources
- **Extended Support**:
  - FMC, SYSCFG, IPCC, HSEM
  - DMA, RTC, TAMP, RCC, PWR
  - EXTI, GPIO

### Address Mapping Strategy

#### Memory Regions
- **Boot ROM**: Fixed address for Cortex-A35 boot code
- **Internal SRAM**: Distributed across domains with different attributes
- **Peripheral Buses**: APB1-4 with structured address allocation
- **External Memory**: DDR space with configurable mappings
- **TrustZone Memory Partitioning**:
  - Secure and non-secure address spaces
  - Hardware-enforced access controls
  - Memory protection unit (MPU) integration

#### Security Partitioning
- **TrustZone Implementation**:
  - Cortex-A35: Arm TrustZone technology
  - Cortex-M33: Armv8-M TrustZone
  - Peripheral attribution: Secure/non-secure configuration
- **Hardware Security**:
  - Active tamper detection
  - Environmental monitoring
  - Cryptographic acceleration with SCA protection

## Power Scheme

### Power Domains
- **VDD**: Main I/O supply (1.71-1.95V or 2.7-3.6V)
- **VDDCORE**: Digital core supply
- **VDDCPU**: Cortex-A35 supply
- **VDDGPU**: GPU/NPU supply (optional)
- **VBAT**: Backup domain supply (1.55-3.6V)
- **VDDIO1/2/3/4**: Separate I/O supplies for different interfaces
- **VDDA18***: Multiple 1.8V analog supplies

### Power Management
- **Multiple low-power modes**:
  - CSleep (CPU clock stopped)
  - CStop (CPU subsystem clock stopped) 
  - DStop1/2 (domain clock stopped)
  - DStandby (domain power down)
  - Stop1/2, LP-Stop1/2, LPLV-Stop1/2
  - Standby1/2 (system powered down)
- **DDR retention** in Standby mode
- **Dynamic Voltage Frequency Scaling (DVFS)**
- **Power switches** for RETRAM, BKPSRAM, SmartRun domains

### Always-On Domain
- **Backup domain**: RTC, tamper, backup registers, LSE, LSI
- **SmartRun domain**: Cortex-M0+, LPSRAM, low-power peripherals

## Top-level I/O

### Critical Interfaces
- **DDR**: 16/32-bit data, address/command/control signals
- **Ethernet**: RGMII/MII/RMII (up to 3 interfaces)
- **USB**: Multiple differential pairs (HS/SS)
- **PCIe**: Differential serial lane
- **Display**: MIPI DSI (4 lanes), LVDS (dual-link)
- **Camera**: MIPI CSI-2 (2 lanes), parallel interfaces
- **OCTOSPI**: 8-bit serial flash interfaces (2x)

### GPIO
- **Up to 172 secure I/O ports** with interrupt capability
- **6 wake-up inputs**
- **8 tamper inputs + 8 active tamper outputs**
- **Multiple supply domains** (VDDIO1-4)
- **High-current capable** with speed selection

### Clock and Reset
- **HSE, LSE** crystal interfaces
- **NRST** (reset input/output)
- **Multiple clock outputs** for external devices

## Key Constraints and Strategies

### Timing Domain Crossings
- Multiple clock domain synchronizers throughout the design
- Asynchronous FIFOs for data transfer between clock domains
- Clock gating controllers for power management

### Isolation and Level Conversion
- TrustZone security isolation for memory and peripherals
- Voltage level shifters between different supply domains
- Resource isolation framework (RIF) for hardware compartmentalization

### Low-Power Strategy
- Hierarchical clock gating at module and sub-module levels
- Power domain partitioning with independent control
- Autonomous operation in low-power modes for critical functions
- Wake-up event management from multiple sources
- DDR self-refresh and retention modes

### Security Constraints
- Secure and non-secure worlds separation with hardware enforcement
- Tamper detection and response with automatic data erasure
- Cryptographic acceleration with side-channel attack protection
- Secure boot and OTP fuse management with anti-rollback protection


### Memory Subsystem
- **Internal SRAM**: Total 808KB
  - SYSRAM: 256KB (AXI, MPU domain) - half/full hardware erase on reset
  - VDERAM: 128KB (video RAM, can be used as general purpose when VDEC/VENC not used)
  - SRAM1: 128KB (AHB, MCU domain) - hardware erase on tamper detection
  - SRAM2: 128KB (AHB, MCU domain)
  - RETRAM: 128KB (retention, ECC/CRC protected) - retention in Standby/VBAT mode
  - BKPSRAM: 8KB (backup, tamper protected, ECC)
  - LPSRAM1/2/3: 8KB+8KB+16KB (SmartRun domain) - retention in Standby/VBAT mode

- **External SDRAM**: Up to 4GB
  - DDR3L: 16/32-bit, up to 2133 MT/s (1066 MHz clock)
  - DDR4: 16/32-bit, up to 2400 MT/s (1200 MHz clock)
  - LPDDR4: 16/32-bit, up to 2400 MT/s (1200 MHz clock)

## Memory mapping

### Cortex-A35 memory map

| Base address | Size | Region name | Description | Comment |
|-------------|------|-------------|-------------|---------|
| 0x0000_0000 | 128 KB | ROM | Boot ROM | Secure, Cortex-A35 only |
| 0x1000_0000 | 256 KB | SYSRAM | System RAM | MPU domain, configurable security |
| 0x1004_0000 | 128 KB | VDERAM | Video RAM | Can be used as general purpose when VDEC/VENC not used |
| 0x2000_0000 | 128 KB | SRAM1 | MCU SRAM 1 | AHB, MCU domain |
| 0x2002_0000 | 128 KB | SRAM2 | MCU SRAM 2 | AHB, MCU domain |
| 0x2004_0000 | 128 KB | RETRAM | Retention RAM | ECC/CRC protected, retention in Standby/VBAT |
| 0x4000_0000 | 32 KB | APB1 | Advanced peripheral bus 1 | Basic peripherals |
| 0x4000_8000 | 32 KB | APB2 | Advanced peripheral bus 2 | Advanced peripherals |
| 0x4001_0000 | 32 KB | APB3 | Advanced peripheral bus 3 | System peripherals |
| 0x4001_8000 | 32 KB | APB4 | Advanced peripheral bus 4 | Security peripherals |
| 0x5000_0000 | 1 MB | AHB1 | Advanced high-performance bus 1 | GPIOs and basic peripherals |
| 0x5001_0000 | 1 MB | AHB2 | Advanced high-performance bus 2 | Connectivity peripherals |
| 0x5002_0000 | 1 MB | AHB3 | Advanced high-performance bus 3 | Analog and timers |
| 0x5003_0000 | 1 MB | AHB4 | Advanced high-performance bus 4 | System functions |
| 0x5004_0000 | 1 MB | AHB5 | Advanced high-performance bus 5 | Camera and display |
| 0x5005_0000 | 1 MB | AHB6 | Advanced high-performance bus 6 | External interfaces |
| 0x6000_0000 | 256 MB | FMC | Flexible memory controller | NOR/PSRAM/NAND Flash |
| 0x7000_0000 | 256 MB | QUADSPI | Quad-SPI Flash interface | Serial NOR/NAND Flash |
| 0x8000_0000 | 2 GB | DDR | DDR SDRAM | DDR3L/DDR4/LPDDR4, up to 4GB |

### Cortex-M33 memory map

| Base address | Size | Region name | Description | Comment |
|-------------|------|-------------|-------------|---------|
| 0x0000_0000 | 128 KB | ROM | Boot ROM | Not accessible from Cortex-M33 |
| 0x1000_0000 | 256 KB | SYSRAM | System RAM | MPU domain, shared with Cortex-A35 |
| 0x1004_0000 | 128 KB | VDERAM | Video RAM | Shared with Cortex-A35 |
| 0x2000_0000 | 128 KB | SRAM1 | MCU SRAM 1 | AHB, MCU domain, primary M33 RAM |
| 0x2002_0000 | 128 KB | SRAM2 | MCU SRAM 2 | AHB, MCU domain |
| 0x2004_0000 | 128 KB | RETRAM | Retention RAM | ECC/CRC protected |
| 0x4000_0000 | 32 KB | APB1 | Advanced peripheral bus 1 | Basic peripherals |
| 0x4000_8000 | 32 KB | APB2 | Advanced peripheral bus 2 | Advanced peripherals |
| 0x4001_0000 | 32 KB | APB3 | Advanced peripheral bus 3 | System peripherals |
| 0x4001_8000 | 32 KB | APB4 | Advanced peripheral bus 4 | Security peripherals |
| 0x5000_0000 | 1 MB | AHB1 | Advanced high-performance bus 1 | GPIOs and basic peripherals |
| 0x5001_0000 | 1 MB | AHB2 | Advanced high-performance bus 2 | Connectivity peripherals |
| 0x5002_0000 | 1 MB | AHB3 | Advanced high-performance bus 3 | Analog and timers |
| 0x5003_0000 | 1 MB | AHB4 | Advanced high-performance bus 4 | System functions |
| 0x8000_0000 | 2 GB | DDR | DDR SDRAM | Shared with Cortex-A35 |

### Backup domain memory map

| Base address | Size | Region name | Description | Comment |
|-------------|------|-------------|-------------|---------|
| 0x5400_0000 | 8 KB | BKPSRAM | Backup SRAM | Tamper protected, ECC |
| 0x5400_2000 | 512 B | BKPREG | Backup registers | 128 × 32-bit registers |
| 0x5400_2400 | 1 KB | TAMP | Tamper and backup control | Tamper detection configuration |

### SmartRun domain memory map

| Base address | Size | Region name | Description | Comment |
|-------------|------|-------------|-------------|---------|
| 0x5402_0000 | 8 KB | LPSRAM1 | Low-power SRAM 1 | Retention in Standby/VBAT |
| 0x5402_2000 | 8 KB | LPSRAM2 | Low-power SRAM 2 | Retention in Standby/VBAT |
| 0x5402_4000 | 16 KB | LPSRAM3 | Low-power SRAM 3 | SmartRun domain |
| 0x5402_8000 | 4 KB | APBSR | SmartRun APB | Low-power peripherals |

## Peripheral register mapping

### APB1 peripherals (0x4000_0000 - 0x4000_7FFF)

| Base address | Peripheral | Description |
|-------------|------------|-------------|
| 0x4000_0000 | TIM2 | 32-bit general-purpose timer |
| 0x4000_0400 | TIM3 | 32-bit general-purpose timer |
| 0x4000_0800 | TIM4 | 32-bit general-purpose timer |
| 0x4000_0C00 | TIM5 | 32-bit general-purpose timer |
| 0x4000_1000 | TIM6 | 16-bit basic timer |
| 0x4000_1400 | TIM7 | 16-bit basic timer |
| 0x4000_8000 | I2C1 | Inter-integrated circuit 1 |
| 0x4000_8400 | I2C2 | Inter-integrated circuit 2 |
| 0x4000_8800 | I2C3 | Inter-integrated circuit 3 |
| 0x4000_8C00 | I2C4 | Inter-integrated circuit 4 |
| 0x4000_9000 | I2C5 | Inter-integrated circuit 5 |
| 0x4000_9400 | I2C6 | Inter-integrated circuit 6 |
| 0x4000_9800 | I2C7 | Inter-integrated circuit 7 |
| 0x4000_9C00 | I2C8 | Inter-integrated circuit 8 |
| 0x4000_A000 | USART2 | Universal synchronous/asynchronous receiver/transmitter 2 |
| 0x4000_A400 | USART3 | Universal synchronous/asynchronous receiver/transmitter 3 |
| 0x4000_A800 | UART4 | Universal asynchronous receiver/transmitter 4 |
| 0x4000_AC00 | UART5 | Universal asynchronous receiver/transmitter 5 |
| 0x4000_B000 | SPI2 | Serial peripheral interface 2 |
| 0x4000_B400 | SPI3 | Serial peripheral interface 3 |

### APB2 peripherals (0x4000_8000 - 0x4000_FFFF)

| Base address | Peripheral | Description |
|-------------|------------|-------------|
| 0x4000_8000 | TIM1 | 16-bit advanced-control timer |
| 0x4000_8400 | TIM8 | 16-bit advanced-control timer |
| 0x4000_8800 | TIM20 | 16-bit advanced-control timer |
| 0x4000_8C00 | TIM15 | 16-bit general-purpose timer |
| 0x4000_9000 | TIM16 | 16-bit general-purpose timer |
| 0x4000_9400 | TIM17 | 16-bit general-purpose timer |
| 0x4000_9800 | USART1 | Universal synchronous/asynchronous receiver/transmitter 1 |
| 0x4000_9C00 | USART6 | Universal synchronous/asynchronous receiver/transmitter 6 |
| 0x4000_A000 | UART7 | Universal asynchronous receiver/transmitter 7 |
| 0x4000_A400 | UART8 | Universal asynchronous receiver/transmitter 8 |
| 0x4000_A800 | UART9 | Universal asynchronous receiver/transmitter 9 |
| 0x4000_AC00 | SPI1 | Serial peripheral interface 1 |
| 0x4000_B000 | SPI4 | Serial peripheral interface 4 |
| 0x4000_B400 | SPI5 | Serial peripheral interface 5 |
| 0x4000_B800 | SPI6 | Serial peripheral interface 6 |
| 0x4000_BC00 | SPI7 | Serial peripheral interface 7 |
| 0x4000_C000 | SPI8 | Serial peripheral interface 8 |
| 0x4000_C400 | SAI1 | Serial audio interface 1 |
| 0x4000_C800 | SAI2 | Serial audio interface 2 |
| 0x4000_CC00 | SAI3 | Serial audio interface 3 |
| 0x4000_D000 | SAI4 | Serial audio interface 4 |

### APB3 peripherals (0x4001_0000 - 0x4001_7FFF)

| Base address | Peripheral | Description |
|-------------|------------|-------------|
| 0x4001_0000 | SYSCFG | System configuration controller |
| 0x4001_0400 | RCC | Reset and clock controller |
| 0x4001_0800 | PWR | Power control |
| 0x4001_0C00 | TAMP | Tamper and backup registers |
| 0x4001_1000 | RTC | Real-time clock |
| 0x4001_1400 | CRC | Cyclic redundancy check calculation unit |
| 0x4001_1800 | HSEM | Hardware semaphore |
| 0x4001_1C00 | IPCC1 | Inter-processor communication controller 1 |
| 0x4001_2000 | EXTI1 | Extended interrupts and events controller 1 |
| 0x4001_2400 | DMA1 | Direct memory access controller 1 |
| 0x4001_2800 | DMA2 | Direct memory access controller 2 |

### APB4 peripherals (0x4001_8000 - 0x4001_FFFF)

| Base address | Peripheral | Description |
|-------------|------------|-------------|
| 0x4001_8000 | HASH | Hash processor |
| 0x4001_8400 | PKA | Public key accelerator |
| 0x4001_8800 | RNG | Random number generator |
| 0x4001_8C00 | BSEC | Boot and security OTP control |
| 0x4001_9000 | RIF | Resource isolation framework |
| 0x4001_9400 | EXTI2 | Extended interrupts and events controller 2 |

### AHB1 peripherals (0x5000_0000 - 0x5000_FFFF)

| Base address | Peripheral | Description |
|-------------|------------|-------------|
| 0x5000_0000 | GPIOA | General-purpose I/Os port A |
| 0x5000_0400 | GPIOB | General-purpose I/Os port B |
| 0x5000_0800 | GPIOC | General-purpose I/Os port C |
| 0x5000_0C00 | GPIOD | General-purpose I/Os port D |
| 0x5000_1000 | GPIOE | General-purpose I/Os port E |
| 0x5000_1400 | GPIOF | General-purpose I/Os port F |
| 0x5000_1800 | GPIOG | General-purpose I/Os port G |
| 0x5000_1C00 | GPIOH | General-purpose I/Os port H |
| 0x5000_2000 | GPIOI | General-purpose I/Os port I |
| 0x5000_2400 | GPIOJ | General-purpose I/Os port J |
| 0x5000_2800 | GPIOK | General-purpose I/Os port K |
| 0x5000_2C00 | GPIOZ | General-purpose I/Os port Z |

### AHB2 peripherals (0x5001_0000 - 0x5001_FFFF)

| Base address | Peripheral | Description |
|-------------|------------|-------------|
| 0x5001_0000 | ADC1 | Analog-to-digital converter 1 |
| 0x5001_0400 | ADC2 | Analog-to-digital converter 2 |
| 0x5001_0800 | ADC3 | Analog-to-digital converter 3 |
| 0x5001_0C00 | DCMI | Digital camera interface |
| 0x5001_1000 | DCMIPP | Digital camera interface with pixel processing |
| 0x5001_1400 | MDF1 | Multi-function digital filter |
| 0x5001_1800 | ADF1 | Audio digital filter |
| 0x5001_1C00 | VREFBUF | Voltage reference buffer |

### AHB3 peripherals (0x5002_0000 - 0x5002_FFFF)

| Base address | Peripheral | Description |
|-------------|------------|-------------|
| 0x5002_0000 | SDMMC1 | Secure digital input/output multi-media card interface 1 |
| 0x5002_0400 | SDMMC2 | Secure digital input/output multi-media card interface 2 |
| 0x5002_0800 | SDMMC3 | Secure digital input/output multi-media card interface 3 |
| 0x5002_0C00 | OCTOSPI1 | Octo-SPI memory interface 1 |
| 0x5002_1000 | OCTOSPI2 | Octo-SPI memory interface 2 |
| 0x5002_1400 | FMC | Flexible memory controller |
| 0x5002_1800 | ETH1 | Ethernet MAC 1 |
| 0x5002_1C00 | ETH2 | Ethernet MAC 2 |
| 0x5002_2000 | ETHSW | Ethernet switch |

### AHB4 peripherals (0x5003_0000 - 0x5003_FFFF)

| Base address | Peripheral | Description |
|-------------|------------|-------------|
| 0x5003_0000 | USBH | USB host controller |
| 0x5003_0400 | USB3DR | USB 3.0 dual-role data controller |
| 0x5003_0800 | UCPD1 | USB Type-C power delivery controller |
| 0x5003_0C00 | PCIE | PCI Express interface |
| 0x5003_1000 | LTDC | LCD-TFT display controller |
| 0x5003_1400 | DSI | Display serial interface |
| 0x5003_1800 | LVDS | LVDS display interface |
| 0x5003_1C00 | VENC | Video encoder |
| 0x5003_2000 | VDEC | Video decoder |
| 0x5003_2400 | GPU | Graphics processing unit |
| 0x5003_2800 | NPU | Neural processing unit |

## External memory mapping

### FMC memory banks

| Base address | Size | Bank | Memory type | Description |
|-------------|------|------|-------------|-------------|
| 0x6000_0000 | 64 MB | Bank1 | NOR/PSRAM | Chip select 1 |
| 0x6400_0000 | 64 MB | Bank2 | NOR/PSRAM | Chip select 2 |
| 0x6800_0000 | 64 MB | Bank3 | NOR/PSRAM | Chip select 3 |
| 0x6C00_0000 | 64 MB | Bank4 | NAND Flash | Chip select 4 |

### OCTOSPI memory mapped mode

| Base address | Size | Port | Memory type | Description |
|-------------|------|------|-------------|-------------|
| 0x7000_0000 | 256 MB | Port1 | Serial NOR/NAND | Memory mapped mode |
| 0x8000_0000 | 256 MB | Port2 | Serial NOR/NAND | Memory mapped mode |

### DDR memory region

| Base address | Size | Description | Comment |
|-------------|------|-------------|---------|
| 0x8000_0000 | 2 GB | DDR SDRAM | Configurable security attributes |

## Security memory partitioning

### TrustZone address space controller regions

| Region | Base address | Size | Security attribute | Description |
|--------|-------------|------|-------------------|-------------|
| Region 0 | 0x0000_0000 | 128 KB | Secure | Boot ROM |
| Region 1 | 0x1000_0000 | 384 KB | Configurable | SYSRAM + VDERAM |
| Region 2 | 0x2000_0000 | 384 KB | Configurable | SRAM1 + SRAM2 + RETRAM |
| Region 3 | 0x4000_0000 | 512 MB | Mixed | Peripheral buses |
| Region 4 | 0x8000_0000 | 2 GB | Configurable | DDR SDRAM |

### Resource isolation framework compartments

| Compartment | Memory regions | Peripheral access | Description |
|-------------|----------------|-------------------|-------------|
| Compartment 0 | 0x1000_0000-0x1003_FFFF | APB1-4 peripherals | Secure world |
| Compartment 1 | 0x2000_0000-0x2005_FFFF | Selected AHB peripherals | Normal world |
| Compartment 2-7 | Configurable | Configurable | Application-specific |

