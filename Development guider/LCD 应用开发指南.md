# ESP_LCD 驱动框架介绍

ESP 的 LCD 驱动位于 **ESP-IDF** 下的 [components/esp_lcd](https://github.com/espressif/esp-idf/tree/master/components/esp_lcd)，目前仅存在于 **release/v4.4 及以上**版本中。**esp_lcd** 能够驱动 ESP 系列芯片硬件所支持的 **I2C**、**SPI**、**8080** 以及 **RGB** 四种接口的 LCD 屏幕，各系列芯片所支持的 LCD 接口如下表所示。

|   SoC    |                          I2C 接口                           |                          SPI 接口                           |                          8080 接口                          |                          RGB 接口                           |
| -------- | ----------------------------------------------------------- | ----------------------------------------------------------- | ----------------------------------------------------------- | ----------------------------------------------------------- |
| ESP32    | ![supported](https://img.shields.io/badge/-Supported-green) | ![supported](https://img.shields.io/badge/-Supported-green) | ![supported](https://img.shields.io/badge/-Supported-green) |                                                             |
| ESP32-S2 | ![supported](https://img.shields.io/badge/-Supported-green) | ![supported](https://img.shields.io/badge/-Supported-green) | ![supported](https://img.shields.io/badge/-Supported-green) |                                                             |
| ESP32-S3 | ![supported](https://img.shields.io/badge/-Supported-green) | ![supported](https://img.shields.io/badge/-Supported-green) | ![supported](https://img.shields.io/badge/-Supported-green) | ![supported](https://img.shields.io/badge/-Supported-green) |
| ESP32-C3 | ![supported](https://img.shields.io/badge/-Supported-green) | ![supported](https://img.shields.io/badge/-Supported-green) |                                                             |                                                             |
各接口的 LCD 驱动应用示例参考 ESP-IDF 下的 [examples/peripherals/lcd](https://github.com/espressif/esp-idf/tree/master/examples/peripherals/lcd)，这些示例目前仅存在于 **release/v5.0** 及以上版本中，因为 **release/v4.4** 中 esp_lcd 的 API 名称与高版本基本一致，所以同样可以参考上述示例（两者的 API 实现上有一些区别），**后续均以 ESP-IDF release/5.0 为基础进行介绍**。

由于 RGB LCD 屏幕的驱动原理与其他接口屏幕有本质差异，下面将按照 **非 RGB** 接口和 **RGB** 接口分别进行介绍：

## 非 RGB 接口

### 硬件驱动框架

![非 RGB 硬件驱动框架](./%E9%9D%9E%20RGB%20%E7%A1%AC%E4%BB%B6%E9%A9%B1%E5%8A%A8%E6%A1%86%E6%9E%B6.png)

包含 I2C、SPI 以及 8080 接口，这类接口屏幕上的驱动 IC 使用显存 GRAM，ESP 只需要把显示数据传给驱动 IC，驱动 IC 会把数据保存到显存中，并按照自身的刷新速率把显存中的数据显示到屏幕上。

### 软件驱动框架



# SPI 接口屏幕配置

# 8080 接口屏幕配置

# RGB 接口屏幕配置

# 用户屏幕接口判断