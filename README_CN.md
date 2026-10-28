<h1 align="center">VIEWE 2.1" 360×360 ESP32-S31 智能旋钮显示快速指南</h1>

* **[English](./README.md)**

<p align="center">
    <img src="images/Product_pic0.png" alt="SFG-VIEWE-DesKnob 产品外观">
</p>

> [!NOTE]
> 产品照片与示意图（`images/`）尚未入库。下文路径为预留位置，素材就绪后可直接放入 `images/`。

---

## 1. 简介

**SFG-VIEWE-DesKnob**（GitHub 仓库：**VIEWE-DesKnob**）是 VIEWE 成品系列（**SFG** 前缀）中的高性能 AI 智能旋钮显示模组。主控为乐鑫 **ESP32-S31NRV16**（QFN80，封装内 **16 MB** 1.8 V Octal PSRAM；板载外置 **W25Q128** Flash **16 MB**），搭配 **2.1 英寸**圆形电容触摸屏（分辨率 **360×360**，显示驱动 **GC9B72NA**，触摸 **CST826**，主机侧 **QSPI**）。硬件为 **双 PCB** 架构（**Knob Module Core Board S31v1** + **Knob Module Dock Board S31v1**），经板对板 **FPC2** 互联，核心板与底座板组成完整模组。

产品集成旋转检测（阿尔卑斯 **SSCM** 双向检测开关，网络名 **EC1** / **EC2**）、**12 颗** WS2812 兼容灯环、**ES8311** 音频编解码 + **NS4150B** 功放 + 模拟麦克风、**Type-C** 供电/下载（原生 USB，**无 CH340**）、**TP4055** 锂电充电及 **MicroSD** 扩展。原理图含 **DRV2605L** 触觉反馈，**当前未贴装/未使用**。用户可自制底板扩展应用，典型场景包括智能家居中枢旋钮、语音助手前端、轻量 HMI 等。

> [!NOTE]
> **ESP32-S31** 目前为**预览**芯片。ESP-IDF 编译须带 **`--preview`**（例如 `idf.py --preview set-target esp32s31`）。模组电气与射频额定值以乐鑫 ESP32-S31 系列模组数据手册（预发布）为准。带版本号的稳定版 ESP-IDF（如 5.4、5.5）不含该芯片，需安装 ESP-IDF **master**。需要 Wi-Fi 6、原生 USB 与板载 Codec 音频的新设计可选用本 S31 旋钮方案。

### 1.1 产品特性

**CPU / 存储 / 无线 / 安全：**

- **处理器（ESP32-S31NRV16）**
  - SoC：高性能 HP 系统 + 低功耗 LP 系统。
  - HP：32 位 RISC-V 双核，最高 320 MHz；每核单精度 FPU；一核 SIMD；双核 CoreMark® @320 MHz：2195.20（6.86/MHz）。
  - LP：32 位 RISC-V 单核，最高 40 MHz，2 级流水线；Deep-sleep 下仍可工作。
  - 封装：QFN80（8×8 mm）；芯片环境温度 −40 ~ 85 °C（N 级）。
  - 天线：原理图含板载天线（CA-C03）及可选 IPEX 外置天线座。
- **存储器**
  - 片内：320 KB ROM、512 KB HP SRAM、32 KB LP SRAM。
  - 封装内 PSRAM：**16 MB**，1.8 V Octal SPI（引脚未引出）。本产品为 **NRV16**，勿与 ESP32-S31NRV32（32 MB）混淆。
  - 板载外置 Flash：**W25Q128JVPIQ**（128 Mbit = **16 MB**）。Flash/PSRAM 在 **1.8 V** 域供电（原理图 R4 10K 至 GND 表示 1.8 V Flash；存在 `VDD_PSRAM_1P8_*`）。W25Q128 为外置 Flash，非 PSRAM。
  - 支持 Flash 与 PSRAM 并行访问；外置 Flash 支持 SPI/Dual/Quad/QPI（芯片能力最高 256 MB）。
- **无线**
  - 2.4 GHz Wi-Fi 6（IEEE 802.11ax，仅 20 MHz 非 AP；兼容 b/g/n），1T1R。
  - 蓝牙® 5.4（LE）+ 经典蓝牙。
  - Zigbee 3.0 与 Thread 1.4（IEEE 802.15.4）。
- **安全（芯片级摘要）**
  - Secure Boot；片外存储加解密（Flash/PSRAM XTS-AES）。
  - 硬件加速：AES / SHA / RSA / ECC / HMAC、数字签名外设、TRNG、密钥管理器等。
  - APM、TEE 等（详见乐鑫官方数据手册）。
- **外设接口（【底座】+【核心】）**
  - 【核心】MicroSD（TF-018），**4-bit SDMMC**
  - 【核心】QSPI 显示屏（FPC1）
  - 【核心】旋转检测：2 个双向检测开关（非 EC11 增量编码器）；**EC1** 接 GPIO2/3；**EC2** 经 FPC2
  - 【核心+底座】板对板 **FPC2**（仅核心板时可作客户扩展）
  - 【底座】USB-C：5 V 供电、固件下载与调试；USB 2.0 High-Speed OTG + USB Serial/JTAG，**无 CH340**
  - 【底座】TP4055 单节锂电充电，PROG 电阻 2.2 kΩ，充电电流约 **455 mA**（§2.9 表典型 **≈470 mA**）
  - 【底座】侧面滑动电源开关 + BOOT 下载键
  - 【底座】12 颗 RGB 灯环（WS2812 兼容）
  - 【底座】ES8311 + NS4150B + ZTS6216 模拟 MIC + 喇叭接口 **CN2**

**显示屏：**

- 尺寸：2.1 英寸（圆形）
- 分辨率：360 × 360
- 显示模组料号：**ALL-UE021HV-RU39-A021A**
- 接口：**QSPI**（核心板实际走线，非并行 RGB）
- 驱动 IC：GC9B72NA
- 触摸 IC：CST826（多点电容，G+F CTP）
- 亮度：**300（min）/ 350（typ）cd/m²**
- 逻辑电源：3.3 V

**其他：**

- 工作温度：−20 ~ 70 °C（受液晶限制）
- 存储温度：−30 ~ 80 °C
- MCU 模组额定温度：−40 ~ 85 °C

### 1.2 应用领域

具备无线、圆形触摸、旋转交互与本地音频能力，适用于：

- 智能家居中枢旋钮 / 情景面板
- 音箱、灯光、空调等设备控制器
- 会议桌 / 桌面办公控制器
- 工业轻量 HMI / 参数旋钮
- AI 语音助手前端（本地 MIC + 喇叭）
- 创客、教学与原型平台

### 1.3 型号含义

官方品名：**SFG-VIEWE-DesKnob**

| 字段 | 代码 | 含义 |
| --- | --- | --- |
| 形态 | SFG | 半成品（Semi-Finished Goods） |
| 品牌 | VIEWE | VIEWE 品牌 |
| 结构特征 | DesKnob | Desk + Knob，桌面旋钮设备 |

---

## 2. 产品信息

### 2.1 接口说明

![接口布局](images/Interface_Layout_cn.jpg)

外形为圆柱形智能旋钮：顶部 2.1" 圆形触摸玻璃，中部半透明灯环，侧面 Type-C、滑动电源开关、麦克风与喇叭开孔。内部堆叠（由后向前）：后壳 → 锂电与喇叭 → 底座板 → 导光环 → 支架 → 旋转轴承 → 金属外壳 → 核心板 → 2.1" 圆形触摸屏（见 §4）。

1. **主控：** ESP32-S31NRV16（HP 双核 RISC-V 最高 320 MHz + LP 单核），封装内 16 MB PSRAM + 板载 W25Q128（16 MB）Flash，板载 / IPEX 天线。
2. **显示与触摸（FPC1）：** 40 针 **FPC-05F-40PH20**，QSPI 屏 + I2C 触摸（`TP_SCL` / `TP_SDA` / `TP_RST` / `TP_INT`）；背光 **LCD_BLK** 经 AO3400A（Q2）。见下表。
3. **SD 卡槽：** 核心板外置存储，**4-bit SDMMC**（§2.8）。
4. **旋转 EC1：** 核心板双向检测开关，**GPIO2 / GPIO3**，10 kΩ 上拉（§2.7）。
5. **旋转 EC2：** 第二路检测经 **FPC2**（§2.5）。
6. **外置天线：** 板载天线，可选 IPEX。
7. **定位柱**（机械结构）。
8. **FPC2：** 核心↔底座互联及客户扩展（§2.5）。
9–11. **【底座】音频：** ES8311 + NS4150B + MIC（ZTS6216）+ 喇叭 **CN2**（1.25-2P）。§2.4。
12. **【底座】电池 CN1：** 1.25-2P，单节锂电 **3.7 V**，**750 mAh**。§2.8。
13. **【底座】电源滑动开关 SW3：** MST-12D18G4，控制 **VCC_OUT**（§2.7）。
14. **【底座】USB Type-C：** 5 V 与原生 USB 下载/调试（§2.6）。
15. **【底座】BOOT 键：** 底座 **SW1**（GT-TC018B-H035-L1），拉低 **BOOTMODE（GPIO61）** 进入下载（§2.7）。*不是*核心板上同为 SW1 的 SSCM 开关。
16. **【底座】RGB 灯环：** 12× XL-1010RGBC-2812B，数据 **RGB_DATA（GPIO61）**（§2.7）。

#### 双板架构

| 板卡 | 丝印/原理图名称 | 主要功能 |
| --- | --- | --- |
| 核心板 | Knob Module Core Board S31v1 | MCU、显示/触摸 FPC、旋转 EC1（及 EC2 引出）、MicroSD、3V3 Buck、FPC2 |
| 底座板 | Knob Module Dock Board S31v1 | Type-C、充电、电源开关、音频、灯环、BOOT、FPC2 |

#### 显示接口（FPC1，核心板 QSPI）

连接器：**FPC-05F-40PH20**（40 针）。下表为核心板原理图已引出信号（勿从屏体机械图复制并行 DB 引脚定义）。

| 引脚 | 符号 | I/O | 说明 |
| --- | --- | --- | --- |
| 2 | TP_RST | I/O | 触摸复位 → GPIO8 |
| 3 | TP_SCL | I/O | 触摸 I2C SCL → GPIO9（原理图与 HAPTIC 网络共用） |
| 4 | TP_SDA | I/O | 触摸 I2C SDA → GPIO10（原理图与 HAPTIC 网络共用） |
| 5 | TP_INT | I | 触摸中断 → GPIO11 |
| 16 | LCD_RST | O | LCD 复位 → GPIO12 |
| 26 | LCD_QSPI_D3 | I/O | QSPI 数据 3 → GPIO13 |
| 27 | LCD_QSPI_D2 | I/O | QSPI 数据 2 → GPIO14 |
| 28 | LCD_QSPI_D1 | I/O | QSPI 数据 1 → GPIO15 |
| 29 | LCD_QSPI_D0 | I/O | QSPI 数据 0 → GPIO16 |
| 32 | LCD_QSPI_CS | O | QSPI 片选 → GPIO17 |
| 33 | LCD_QSPI_SCL | O | QSPI 时钟 → GPIO18 |
| 35 | LCD_TE | I | 撕裂信号 → GPIO19 |
| 38 | LEDK | P | 背光阴极（经 R30=10 Ω + Q2 AO3400A 至 GND） |
| 39 | LEDA | P | 背光阳极（接 3V3） |
| 其他 | GND/3V3/NC | — | 依显示模组与核心板走线 |

*I：输入；O：输出；P：电源。* 背光使能 **LCD_BLK = GPIO7**（高有效；栅极 10 kΩ 下拉）。

#### 触摸接口摘要

| 信号 | GPIO | 说明 |
| --- | --- | --- |
| TP_RST | GPIO8 | 触摸复位 |
| TP_SCL | GPIO9 | I2C SCL，2.2 kΩ 上拉至 3V3 |
| TP_SDA | GPIO10 | I2C SDA，2.2 kΩ 上拉至 3V3 |
| TP_INT | GPIO11 | 触摸中断 |
| CST826 | — | I2C 地址 **0x15**（规格书） |

### 2.2 GPIO 定义

![GPIO 定义](images/GPIO_Definition.png)

| GPIO | 当前用途 | 功能说明 |
| --- | --- | --- |
| GPIO0–1 | 保留 | 未分配 |
| GPIO2 | EC1_B | 旋转检测通道 B（核心板） |
| GPIO3 | EC1_A | 旋转检测通道 A（核心板） |
| GPIO4–6 | 保留 | 未分配 |
| GPIO7 | LCD_BLK | 背光使能（AO3400A 栅极），高有效 |
| GPIO8 | TP_RST | 触摸复位 |
| GPIO9 | TP_SCL | I2C 时钟（触摸） |
| GPIO10 | TP_SDA | I2C 数据（触摸） |
| GPIO11 | TP_INT | 触摸中断 |
| GPIO12 | LCD_RST | LCD 复位 |
| GPIO13 | LCD_QSPI_D3 | QSPI 数据 3 |
| GPIO14 | LCD_QSPI_D2 | QSPI 数据 2 |
| GPIO15 | LCD_QSPI_D1 | QSPI 数据 1 |
| GPIO16 | LCD_QSPI_D0 | QSPI 数据 0 |
| GPIO17 | LCD_QSPI_CS | QSPI 片选 |
| GPIO18 | LCD_QSPI_SCL | QSPI 时钟 |
| GPIO19 | LCD_TE | LCD TE |
| GPIO20–23 | SD-D0–D3 | SDMMC 4-bit 数据 |
| GPIO24 | SD-CLK | SDMMC 时钟 |
| GPIO25 | SD-CMD | SDMMC 命令 |
| USB_DP / USB_DM | USB 2.0 HS PHY | 模组原生 USB 差分，经 FPC2 至 Type-C |
| GPIO33 | USB_DN | 经 FPC2 至 Type-C；与 ESP_USB_* 共存 |
| GPIO34 | USB_DP | 经 FPC2 至 Type-C；与 ESP_USB_* 共存 |
| GPIO35 | AUDIO_I2C_SCL | ES8311 I2C SCL（底座） |
| GPIO37 | FPC2 引出 | 候选外设 |
| GPIO38 | FPC2 引出 | 候选外设 |
| GPIO39 | FPC2 引出 | 候选外设 |
| GPIO40 | AUDIO_I2C_SDA | ES8311 I2C SDA |
| GPIO42 | I2S_MCLK | I2S 主时钟 |
| GPIO43 | I2S_BCK | I2S 位时钟 |
| GPIO44 | I2S_DI | I2S 数据入（ADC → MCU） |
| GPIO45 | I2S_WS | I2S 字选择 / LRCK |
| GPIO46 | I2S_DO | I2S 数据出（MCU → DAC） |
| GPIO47 | PA_CTRL | NS4150B 功放使能 |
| GPIO48–50 | FPC2 引出 | 候选外设 |
| GPIO51 | BAT_ADC | 电池/系统电压 ADC（分压采样） |
| GPIO52–59 | 保留 | 未分配 |
| GPIO60 | SD-CTRL | SD 3.3 V 电源开关，低有效 |
| GPIO61 | **BOOT / RGB_DATA** | BOOT 键拉低 **BOOTMODE**；同一网络驱动灯环 **RGB_DATA** |

> [!NOTE]
> **GPIO61** 在底座 **BOOT** 键（**BOOTMODE**）与 WS2812 兼容 **RGB_DATA** 之间共用。下载需拉低 GPIO61 并接 Type-C；驱动灯环时需考虑与下载模式的冲突（见常见问题）。

### 2.3 显示参数

| 项目 | 规格 | 单位 | 备注 |
| --- | --- | --- | --- |
| 显示类型 | 圆形 TFT，透射式，常黑 | — | Transmissive, Normally Black |
| 屏幕尺寸 | 2.1 | 英寸 | 对角线 |
| 分辨率 | 360 × 360 | 点 | — |
| 驱动 IC | GC9B72NA | — | — |
| 触摸 IC | CST826 | — | 多点，G+F CTP |
| 主机接口 | QSPI | — | 核心板 FPC1 走线 |
| 模组 OD（含 TP） | 62.80 ± 0.1 | mm | — |
| 可视区 VA | 53.88 ± 0.2 | mm | — |
| 有效区 AA | 52.92 | mm | — |
| 厚度 | 3.71 ± 0.2 | mm | — |
| 亮度 | 300（min）/ 350（typ） | cd/m² | — |
| 视角 | 80/80/80/80 typ. | ° | — |
| 逻辑电源 VDD | 3.3 | V | — |
| 背光 | 4× 白光 LED | — | If≈80 mA，Vf≈2.8 V typ. |
| 工作温度 | −20 ~ 70 | °C | — |

### 2.4 音频接口

**【依赖底座】** 底座上 ES8311 + NS4150B + ZTS6216 模拟 MIC。仅核心板时需经 FPC2 实现 I2S / AUDIO_I2C / PA_CTRL 等。Codec 电源 **CODEC_3V3** 由 5 V 经 RT9013-33GB 产生。喇叭 **CN2**（1.25-2P WT）。

| 信号 | GPIO / 器件 | 说明 |
| --- | --- | --- |
| I2C SDA / SCL | GPIO40 / GPIO35 | ES8311 I2C |
| I2S_MCLK | GPIO42 | I2S 主时钟 |
| I2S_BCK | GPIO43 | 位时钟 |
| I2S_DI | GPIO44 | ADC → MCU |
| I2S_WS | GPIO45 | LRCK / WS |
| I2S_DO | GPIO46 | MCU → DAC |
| PA_EN | GPIO47 | 高电平打开放大器 |
| MIC | ZTS6216 | 差分 ADC_MIC1P/N |
| 喇叭 | NS4150B | 差分输出，典型 4 Ω / 3 W 级 |

### 2.5 板对板 FPC2（客户扩展）

连接线缆规格 **0.3×31P×100 mm**。核心板将空闲 IO 引出供载板使用（电源、USB、音频、LED 数据、BOOT、BAT_ADC、EC2 等）。引脚与核心/底座原理图一致。

| 引脚 | 符号 | 说明 |
| --- | --- | --- |
| 1 | GND | 地 |
| 2 | 5V | 板间 5 V |
| 3 | BOOTMODE | 下载模式（→ GPIO61） |
| 4 | GND | 地 |
| 5 | GPIO49 | 至底座 / 候选外设 |
| 6 | GND | 地 |
| 7 | GPIO48 | 候选外设 |
| 8 | GND | 地 |
| 9 | GPIO47 | PA_CTRL |
| 10 | GPIO51 | BAT_ADC |
| 11 | GPIO46 | I2S_DO |
| 12 | GPIO50 | 候选外设 |
| 13 | GPIO45 | I2S_WS |
| 14 | GPIO39 | 候选外设 |
| 15 | GPIO44 | I2S_DI |
| 16 | GPIO38 | 候选外设 |
| 17 | GPIO43 | I2S_BCK |
| 18 | GPIO37 | 候选外设 |
| 19 | GPIO42 | I2S_MCLK |
| 20 | EC2_B | 第二路旋转检测 B（SSCM，核心 SW1） |
| 21 | GPIO40 | AUDIO_I2C_SDA |
| 22 | EC2_A | 第二路旋转检测 A（SSCM，核心 SW1） |
| 23 | GPIO35 | AUDIO_I2C_SCL |
| 24 | GND | 地 |
| 25 | USB_DN | USB 差分 − |
| 26 | USB_DP | USB 差分 + |
| 27 | GND | 地 |
| 28 | ESP_USB_P | 模组 USB + |
| 29 | ESP_USB_N | 模组 USB − |
| 30 | 5V | 板间 5 V |
| 31–33 | GND | 地 |

### 2.6 USB 与下载

**【依赖底座】** Type-C、ESD、CC 电阻与 BOOT 在底座。整机**无 CH340** 等 USB-UART 桥，下载与枚举依赖 **ESP32-S31 原生 USB**。

| 端口 | 桥接 / PHY | 作用 | 备注 |
| --- | --- | --- | --- |
| Type-C | USB_DP/DN、USB 2.0 HS PHY、ESP_USB_P/N | 下载 / 日志 / 5 V 输入 | CC1/CC2 对地 5.1 kΩ（UFP）；差分线 H5VL10B ESD；**按住 BOOT（GPIO61）** 后接 Type-C |

原理图在 Type-C 上同时引出 **ESP_USB_P/N** 与 **USB_DP/DN**（对应插头 A/B 侧）。软件上均视为模组原生 USB。建议：按住 BOOT → 接 Type-C → `idf.py --preview flash monitor`。

### 2.7 旋转检测 / 按键 / 灯环

旋转机构**不是** EC11 类正交编码器，而是两颗阿尔卑斯 **SSCM110100** 双向检测开关。网络名 **EC1_A/B**、**EC2_A/B** 易被误读为编码器通道，本品为**检测开关**。

| 位号（核心） | 料号 | 网络 | 作用 |
| --- | --- | --- | --- |
| SW2 | SSCM110100 | EC1_A / EC1_B | 主路径至 MCU GPIO2/3 |
| SW1 | SSCM110100 | EC2_A / EC2_B | 第二路径经 FPC2 至底座 |

**位号说明：** 核心 **SW1** = SSCM 检测开关；底座 **SW1** = BOOT 轻触键 **GT-TC018B-H035-L1**。位号相同、器件不同。

**SSCM110100（摘要）：** 1 极 2 位；4.0×5.0×1.5 mm；行程 1.57 mm；操作力最大 0.35 N；额定 1 mA/5 V DC（最大）、50 µA/3 V DC（最小）；寿命 50000 次；−10 ~ +60 °C。软件须处理触点抖动。

**按键与电源（底座）：**

| 名称 | 位置 | 功能 |
| --- | --- | --- |
| BOOT（底座 SW1） | 底座 | GT-TC018B-H035-L1，拉低 BOOTMODE（GPIO61） |
| 电源滑动 SW3 | 底座侧面 | MST-12D18G4，控制 VCC_OUT |

**灯环（底座）：** 12× **1010RGBC-2812B** 串联；**RGB_DATA = GPIO61**，经 R54 = 100 Ω；供电 **VDD_RGB / VCC_OUT**。

### 2.8 存储与充电

**存储**

| 项目 | 说明 |
| --- | --- |
| 板载 Flash | 16 MB，1.8 V 域 |
| PSRAM | 封装内 16 MB 1.8 V Octal SPI（ESP32-S31NRV16） |
| MicroSD | SDMMC 4-bit：CLK=GPIO24，CMD=GPIO25，D0–D3=GPIO20–23；**SD-CTRL=GPIO60**（3.3 V 开关，低有效） |

**电池与充电（底座）**

| 项目 | 说明 |
| --- | --- |
| 充电 IC | TP4055（U1） |
| 电池座 | CN1，1.25-2P WT（BAT+） |
| 电源路径 | 滑动开关 SW3 选择/接通 VCC_OUT |
| 电压采样 | BAT_ADC = **0.4 × VCC_OUT** → GPIO51 |
| 标称电压 | 3.7 V |
| 电池容量 | 750 mAh |

### 2.9 电压与电流

| 项目 | 条件 | 最小 | 典型 | 最大 | 单位 |
| --- | --- | --- | --- | --- | --- |
| 输入电压（USB） | Type-C VBUS | — | 5.0 | — | V |
| 核心逻辑 | TLV62569 输出 | — | 3.3 | — | V |
| 充电电流 | TP4055，Rprog=2.2 kΩ | — | ≈470 | — | mA |
| 系统工作电流 | 亮屏 / Wi-Fi / 音频，5 V | — | 600 | — | mA |
| 系统待机电流 | 背光关、射频空闲 | — | 500 | — | mA |
| 背光 LED 电流 | 模组 4 LED 规格 | — | ≈80 | — | mA |

推荐供电：Type-C **5 V / ≥1 A** 适配器（充电与满载同时运行时留余量）。

### 2.10 可靠性试验（摘要）

| 项目 | 条件 | 最小 | 典型 | 最大 | 单位 |
| --- | --- | --- | --- | --- | --- |
| 工作温度 | 5 V、60% RH | −20 | 25 | 70 | °C |
| 存储温度 | — | −30 | 25 | 85 | °C |
| 工作湿度 | 25 °C | 10% | 60% | 90% | RH |
| ESD | — | — | — | 接触 ±4 kV / 空气 ±8 kV | kV |

---

## 3. 功能框图

![功能框图](images/Functional_Block_Diagram.png)

> **注意：** ESP32-S31 的 2.4 GHz 射频支持 Wi-Fi 6、蓝牙 5.4（LE）、经典蓝牙与 802.15.4，共用同一射频前端，Wi-Fi 与蓝牙不能同时收发，协议间切换使用。USB 2.0 High-Speed 为独立 PHY，不共用该射频前端。

---

## 4. 机械图与爆炸图

![正视 / 侧视 / 后视](images/Dimension_drawing.jpg)

![爆炸结构](images/Exploded_structure.png)

堆叠顺序（由后向前）：后壳 → 锂电与喇叭 → 底座板 → 导光环 → 支架 → 旋转轴承 → 金属外壳 → 核心板 → 2.1" 圆形触摸屏。

---

## 5. 软件

本仓库**尚未**提供 `examples/` 目录，也未发布 DesKnob 专用 BSP 名称。**ESP-IDF** 示例与文档**规划中（TBD）**。规格书说明相关开源资料仍在完善中。

示例发布后预期与 VIEWE 其他 S31 产品一致：

- 安装 ESP-IDF **master**（可用 [EIM](https://dl.espressif.com/dl/eim/index.html)）。带版本号稳定版不含 `esp32s31`。
- 所有目标与烧录命令须带 **`--preview`**。

### 5.1 入门（ESP-IDF 预览目标）

1. **安装 ESP-IDF master**
   - 确认工具链：

```powershell
idf.py --version
idf.py --preview --list-targets
```

   - 版本路径应含 `master`；目标列表须含 `esp32s31`。
2. **设定目标并烧录（待工程发布后）**

```powershell
idf.py --preview set-target esp32s31
idf.py --preview -p PORT flash monitor
```

   - 将 `PORT` 换成本机 **原生 USB** 设备名（不是 CH340 串口）。
   - 建议下载顺序：按需打开电源 → **按住 BOOT** → 接 Type-C → 枚举成功后松开 BOOT → 烧录。

目标列表无 `esp32s31` 时，多为未装 master 或未加 `--preview`。

---

## 6. 相关文档

- [产品规格书 V1.0（PDF）](datasheet/SFG-VIEWE-DesKnob_V1.0_SPEC.pdf)
- 乐鑫 **ESP32-S31** 预发布 / 系列文档（见乐鑫官网；预览芯片以预发布手册为准）

---

## 7. 常见问题

* **Q.** 接 Type-C 为什么看不到 **CH340** 串口？
* **A.** 整机使用 **ESP32-S31 原生 USB**（USB 2.0 High-Speed PHY + USB Serial/JTAG），无 USB-UART 桥。请在系统中选择模组对应的 USB 设备，而非外置串口芯片。

* **Q.** 如何进入固件下载模式？
* **A.** 在底座**按住 BOOT**（拉低 **GPIO61 / BOOTMODE**），再插 **Type-C**（5 V）。枚举成功后松开 BOOT，执行 `idf.py --preview flash`（或 `flash monitor`）。规格书亦写可在按住 BOOT 后点按 **RESET**（若板上有复位键）。

* **Q.** `idf.py --list-targets` 里没有 `esp32s31`？
* **A.** ESP32-S31 为**预览**芯片。须安装 ESP-IDF **master**，命令带 **`--preview`**（如 `idf.py --preview --list-targets`、`idf.py --preview set-target esp32s31`）。5.4、5.5 等稳定版不会列出该目标。

* **Q.** **GPIO61** 既是 **BOOT** 又是 **RGB_DATA**，是否正常？
* **A.** 是。规格书 GPIO 表写明 **GPIO61** 同时承担 **BOOTMODE** 与灯环 **RGB_DATA**。规划固件与产测时避免在需要拉低 GPIO61 下载时被灯环数据干扰。

* **Q.** 旋钮是 **EC11 编码器**吗？
* **A.** 不是。旋转检测为阿尔卑斯 **SSCM110100** 双向**检测开关**（EC1 在 GPIO2/3，EC2 经 FPC2），非正交脉冲。软件需做去抖处理。

* **Q.** 核心 **SW1** 与底座 **SW1** 是同一器件吗？
* **A.** 不是。核心 **SW1** 为 **SSCM** 旋转开关（EC2）；底座 **SW1** 为 **BOOT** 键。两板原理图复用了 SW1 位号。

* **Q.** GitHub 上的示例与图纸在哪里？
* **A.** 仓库内容仍在补充。规格书写明图纸与开源资料持续完善，请关注本仓库及 `datasheet/` 下 PDF 更新。
