# LCD 硬件详解

驱动 LCD 的第一步是了解它的接口类型，对于大部分常见的 LCD 驱动 IC，如 ST7789、GC9A01、ILI9341 等，这些芯片本身一般都会支持多种接口（如 SPI、8080 和 RGB），只是屏幕厂商在封装成 LCD 模组的时候会根据客户的要求或者按照惯例只留出其中一种接口（RGB 屏幕通常会也会留出 SPI 接口，参考[资料](https://focuslcds.com/3-wire-spi-parallel-rgb-interface-fan4213/)），如图 1 所示。

<div align=center><img src="./static/gc9a01_block.png" width=600/></div>
<div align=center>图1  GC9A01 功能框图</div>

驱动 IC 最终使用哪种接口是通过 **IM[3:0]** 这 4 个引脚来决定的，如图 2 所示。大部分屏幕在内部已经定义好了 IM[3:0] ，但是也有一些屏幕会预留 IM[3:0] 以及所有的接口引脚，此时可以自行配置。

<div align=center><img src="./static/st7789_interface.png" width=600/></div>
<div align=center>图2  ST7789 接口定义</div>

因此，仅仅知道驱动 IC 无法判断屏幕的接口类型，在这种情况下可以通过原理图结合经验进行判断，如下：

* **（经验）屏幕分辨率与接口对应关系**:

|     小尺寸     |        中尺寸        |     大尺寸     |
| -------------- | -------------------- | -------------- |
| [1x1, 320x240] | （320x240，480x480） | [480x480，+∞） |
| SPI 或 I2C接口 | 8080接口             | RGB接口        |

* **LCD 引脚**：

    * 各接口通用：RST（RESX）复位
    * **SPI**: 通常包含了 CS、SCK（SCL）、SDA（MOSI）、SDO（MISO） 、RS（WR）
    * **8080**: 通常包含了 CS（CSX）、RD（RDX）、WR（WRX）、DC（D/CX）、D[15:0]（D[7:0]）
    * **RGB**: 通常包含了 三线 SPI、HSYNC、VSYNC、PCLK、DE、D[23:0]（D[17:0]）

# SPI 接口

## 接口模式

<div align=center><img src="./static/spi_interface.png" width=600/></div>
<div align=center>图3  SPI 接口配置</div>

### Interface I/II 模式

* 区别在于**数据读取和写入是否仅用一根数据线**，如图 3、4 所示

<div align=center><img src="./static/spi_serial_interface.png" width=600/></div>
<div align=center>图4  SPI Serial Interface I/II 读时序</div>

* **Interface I** 模式表示仅用一根数据线进行读取和写入，此时，SPI 的 **MOSI** 对应于 **SDA**，且没有 **MISO**
* **Interface II** 模式表示用两根数据线分别进行读取和写入，此时，SPI 的 **MOSI** 对应于 **SDA**，SPI 的 **MISO** 对应于 **SDO**
* 通过 `esp_lcd_panel_io_spi_config_t` 的 `.flags.sio_mode` 可以选择对该模式进行设置

### 3-line/4-line 模式

* 区别在于**是否有 DC（RS）信号线**，如图 3、5 所示

<div align=center><img src="./static/spi_3_4-line.png" width=600/></div>
<div align=center>图5  SPI 3/4-line 写时序</div>

* **3-line** 模式表示没有 DC 线来表示当前是命令还是数据，因此需要将 DC 信息作为 1 bit 数据进行传输，该模式有时也成为 **9-bit 模式**
* **4-line** 模式表示具有 DC 线， `esp_lcd` 目前仅支持该模式

# RGB 接口

## 接口模式

RGB 接口具有两种模式，即 **DE** 和 **HV** 模式，它们的区别在于**是否使用 DE 信号线**，可以通过命令进行配置。

* **DE 模式**：需要使用 VSYNC、HSYNC、PCLK（DOTCLK）、DE、D[X:0] 信号线，时序如下图所示。

<div align=center><img src="./static/st7789_rgb_de.png" width=600/></div>
<div align=center>图6  RGB DE 模式时序</div>

* **HV 模式**：需要使用 VSYNC、HSYNC、PCLK（DOTCLK）、D[X:0] 信号线，时序如下图所示。

<div align=center><img src="./static/st7789_rgb_hv.png" width=600/></div>
<div align=center>图7  RGB HV 模式时序</div>

## 色彩（输入数据）类型

通常来说，RGB 接口支持多种色彩（输入数据）类型，包括 **RGB565**、**RGB666**、**RGB888**等，也可以通过命令进行配置，如下图所示。

<div align=center><img src="./static/st7789_rgb_data.png" width=600/></div>
<div align=center>图8  RGB 输入数据类型</div>

Note: 262K=RGB666、65K=RGB565，参考[资料](https://focuslcds.com/color-depth-65k-262k-16-7m-colors/)

因此，**命令配置需要与硬件端口保持一致**，比如某个 RGB LCD 屏幕仅有 D[17:0] 18 位数据线，需要确保命令配置其数据宽度 <= 18 位。

目前仅有 ESP32-S3 支持 RGB 接口，且其仅支持输出 **16-bit RGB565** （或 8-bit RGB888）色彩类型，但是**通过硬件连接可以使其驱动 RGB666 或 RGB888 类型的屏幕**，参考[原理图](https://github.com/espressif/esp-dev-kits/blob/master/docs/_static/schematics/esp32-s3-lcd-ev-board/SCH_ESP32-S3-LCD_Ev_Board_SUB3_V1.0_20220617.pdf)。
