# RGB LCD 应用代码详解

* 详细说明见[文档](https://docs.espressif.com/projects/esp-idf/en/latest/esp32s3/api-reference/peripherals/lcd.html#rgb-interfaced-lcd)
* 下面以常见的 “3-line SPI + RGB” 为例，对代码中各阶段具体的配置参数进行讲解

## 屏幕初始化

本示例采用了 IO 扩展芯片（TCA9554）或芯片 GPIIO 来模拟 SPI 时序，用户也可以通过硬件 SPI 模拟 SPI。

```
/*
 * SPDX-FileCopyrightText: 2023 Espressif Systems (Shanghai) CO LTD
 *
 * SPDX-License-Identifier: CC0-1.0
 */

#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_check.h"
#include "esp_rom_sys.h"

/******************* 请根据实际情况修改 *******************/
#dfeine BSP_IO_EXPANDER_EN              (0)

#if BSP_IO_EXPANDER_EN
#define BSP_LCD_SPI_CS          (IO_EXPANDER_PIN_NUM_1)
#define BSP_LCD_SPI_SCK         (IO_EXPANDER_PIN_NUM_2)
#define BSP_LCD_SPI_SDO         (IO_EXPANDER_PIN_NUM_3)
#define BSP_LCD_RST             (-1)
#else
#define BSP_LCD_SPI_CS          (GPIO_NUM_38)
#define BSP_LCD_SPI_SCK         (GPIO_NUM_40)
#define BSP_LCD_SPI_SDO         (GPIO_NUM_39)
#define BSP_LCD_RST             (-1)
#endif

#define LCD_CONFIG_DATA_LEN_MAX         (52)
/******************* 请根据实际情况修改 *******************/

#if BSP_IO_EXPANDER_EN
#include "esp_io_expander_tca9554.h"`
#else
#include "driver/gpio.h"
#endif

#if BSP_IO_EXPANDER_EN
static esp_io_expander_handle_t io = NULL;
#endif

#define Delay(t)        vTaskDelay(pdMS_TO_TICKS(t))
#define udelay(t)       esp_rom_delay_us(t)
#if BSP_IO_EXPANDER_EN
#define CS(n)           ESP_ERROR_CHECK(esp_io_expander_set_level(io, BSP_LCD_SPI_CS, n))
#define SCK(n)          ESP_ERROR_CHECK(esp_io_expander_set_level(io, BSP_LCD_SPI_SCK, n))
#define SDO(n)          ESP_ERROR_CHECK(esp_io_expander_set_level(io, BSP_LCD_SPI_SDO, n))
#define RST(n)          ESP_ERROR_CHECK(esp_io_expander_set_level(io, BSP_LCD_RST, n))
#else
#define CS(n)           ESP_ERROR_CHECK(gpio_set_level(BSP_LCD_SPI_CS, n))
#define SCK(n)          ESP_ERROR_CHECK(gpio_set_level(BSP_LCD_SPI_SCK, n))
#define SDO(n)          ESP_ERROR_CHECK(gpio_set_level(BSP_LCD_SPI_SDO, n))
#define RST(n)          ESP_ERROR_CHECK(gpio_set_level(BSP_LCD_RST, n))
#endif

/**************************************************************************************************
 *
 * LCD Configuration Function
 *
 **************************************************************************************************/
/**
 * @brief Simulate SPI to write data using io expander
 *
 * @param data: Data to write
 *
 * @return
 *      - ESP_OK: Success, otherwise returns ESP_ERR_xxx
 */
static esp_err_t spi_write(uint16_t data)
{
    for (uint8_t n = 0; n < 9; n++) {
        if (data & 0x0100) {
            SDO(1);
        } else {
            SDO(0);
        }
        data = data << 1;

        SCK(0);
        udelay(10);
        SCK(1);
        udelay(10);
    }

    return ESP_OK;
}

/**
 * @brief Simulate SPI to write LCD command using io expander
 *
 * @param data: LCD command to write
 *
 * @return
 *      - ESP_OK: Success, otherwise returns ESP_ERR_xxx
 */
static esp_err_t spi_write_cmd(uint16_t data)
{
    CS(0);
    udelay(10);

    spi_write((data & 0x00FF));

    udelay(10);
    CS(1);
    SCK(0);
    SDO(0);
    udelay(10);

    return ESP_OK;
}

/**
 * @brief Simulate SPI to write LCD data using io expander
 *
 * @param data: LCD data to write
 *
 * @return
 *      - ESP_OK: Success, otherwise returns ESP_ERR_xxx
 */
static esp_err_t spi_write_data(uint16_t data)
{
    CS(0);
    udelay(10);

    data &= 0x00FF;
    data |= 0x0100;
    spi_write(data);

    udelay(10);
    CS(1);
    SCK(0);
    SDO(0);
    udelay(10);

    return ESP_OK;
}

/**
 * @brief LCD configuration data structure type
 *
 */
typedef struct {
    uint8_t cmd;            // LCD command
    uint8_t data[LCD_CONFIG_DATA_LEN_MAX];       // LCD data
    uint8_t data_bytes;     // Length of data in below data array; 0xFF = end of cmds.
} lcd_config_data_t;

const static lcd_config_data_t LCD_CONFIG_CMD[] = {
/******************* 请根据实际情况修改 *******************/
    {0xf0, {0x55, 0xaa, 0x52, 0x08, 0x00}, 5},
    {0xf6, {0x5a, 0x87}, 2},
    {0x11, {0x00}, 0},
    ...
/******************* 请根据实际情况修改 *******************/

    {0x00, {0x00}, 0xff},
};

/**
 * @brief Configure LCD with specific commands and data
 *
 * @return
 *      - ESP_OK: Success, otherwise returns ESP_ERR_xxx
 *
 */
static esp_err_t lcd_config(void)
{
#if BSP_IO_EXPANDER_EN
    esp_io_expander_set_dir(io, BSP_LCD_SPI_CS, IO_EXPANDER_OUTPUT);
    esp_io_expander_set_dir(io, BSP_LCD_SPI_SCK, IO_EXPANDER_OUTPUT);
    esp_io_expander_set_dir(io, BSP_LCD_SPI_SDO, IO_EXPANDER_OUTPUT);
    if (BSP_LCD_RST != -1) {
        esp_io_expander_set_dir(io, BSP_LCD_RST, IO_EXPANDER_OUTPUT);
    }
    esp_io_expander_set_level(io, BSP_LCD_SPI_CS, 1);
    esp_io_expander_set_level(io, BSP_LCD_SPI_SCK, 1);
    esp_io_expander_set_level(io, BSP_LCD_SPI_SDO, 1);
#else
    uint64_t bit_msk = BIT64(BSP_LCD_SPI_CS) | BIT64(BSP_LCD_SPI_SCK) | BIT64(BSP_LCD_SPI_SDO);
    if (BSP_LCD_RST != -1) {
        bit_msk |= BIT64(BSP_LCD_RST);
    }
    gpio_config_t config = {
        .mode = GPIO_MODE_OUTPUT,
        .pull_down_en = GPIO_PULLDOWN_DISABLE,
        .pull_up_en = GPIO_PULLUP_DISABLE,
        .pin_bit_mask = bit_msk,
    };
    ESP_RETURN_ON_ERROR(gpio_config(&config), TAG, "SPI GPIO config failed");
#endif

    CS(1);
    SCK(1);
    SDO(1);
    if (BSP_LCD_RST != -1) {
        RST(0);
        Delay(10);
        RST(1);
        Delay(120);
    }

    for (uint8_t i = 0; LCD_CONFIG_CMD_1[i].data_bytes != 0xff; i++) {
        BSP_ERROR_CHECK_RETURN_ERR(spi_write_cmd(LCD_CONFIG_CMD_1[i].cmd));
        for (uint8_t j = 0; j < LCD_CONFIG_CMD_1[i].data_bytes; j++) {
            BSP_ERROR_CHECK_RETURN_ERR(spi_write_data(LCD_CONFIG_CMD_1[i].data[j]));
        }
    }

    Delay(120);
    BSP_ERROR_CHECK_RETURN_ERR(spi_write_cmd(0x29));
    Delay(120);

#if !BSP_IO_EXPANDER_EN
    gpio_reset_pin(BSP_LCD_SPI_SCK);
    gpio_reset_pin(BSP_LCD_SPI_SDO);
#endif

    return ESP_OK;
}
```

* 本示例模拟的 SPI 时序为**模式 0**（CPOL=0,CPHA=0），用户需要查阅 LCD 驱动 IC 数据手册来确认

* 使用示例模板主要需要进行如下修改：

    1. `BSP_IO_EXPANDER_EN` 宏：根据是否使用 IO 扩展芯片，否则使用芯片 IO

    2. `LCD_CONFIG_CMD` 数组：根据屏厂给定的配置数据进行修改，同时根据最大字节数修改 `LCD_CONFIG_DATA_LEN_MAX` 宏

    3. `lcd_config()` 函数：可以自行修改来设置特殊操作和命令（如延时等）

* 对于常见 st、ili、gc 系列 LCD 驱动 IC，初始化配置中有两个命令需要注意下，有时颜色显示不正常可能是因为下列命令与硬件不符合导致的：

    1. **COLCTRL（CDh）**：其中 **MDT** 位决定了 **18-bit RGB666** 采用了几根数据线，如下图所示

        <div align=center ><img src="../_static/cmd_control.png" width=600/></div>

        需要结合具体的硬件连接方式进行设置：
        * MDT=0(CDH=00): D[21:16]=R,D[13:8]=G,D[5:0]=B
        * MDT=1(CDH=08): D[17:12]=R,D[11:6]=G,D[5:0]=B

    2. **COLMOD（3Ah）**：其中 **VIPF[2:0]** 位决定了 LCD 的颜色类型（输入数据格式），如下图所示

        <div align=center ><img src="../_static/cmd_colmod.png" width=400/></div>

        需要结合具体的硬件连接方式进行设置：55/50=16-bit(RGB565);66=18-bit(RGB666);77或默认=24-bit(RGB888)

## RGB 接口初始化

```
/*
 * SPDX-FileCopyrightText: 2023 Espressif Systems (Shanghai) CO LTD
 *
 * SPDX-License-Identifier: CC0-1.0
 */

#include "driver/gpio.h"
#include "esp_lcd_panel_io.h"       // 依赖的头文件
#include "esp_lcd_panel_ops.h"
#include "esp_lcd_panel_rgb.h"

/******************* 请根据实际情况修改 *******************/
#define BSP_LCD_VSYNC                   (GPIO_NUM_3)
#define BSP_LCD_HSYNC                   (GPIO_NUM_46)
#define BSP_LCD_DISP                    (GPIO_NUM_NC)
#define BSP_LCD_DE                      (GPIO_NUM_17)
#define BSP_LCD_PCLK                    (GPIO_NUM_9)
#define BSP_LCD_DATA0                   (GPIO_NUM_10)   // B3
#define BSP_LCD_DATA1                   (GPIO_NUM_11)   // B4
#define BSP_LCD_DATA2                   (GPIO_NUM_12)   // B5
#define BSP_LCD_DATA3                   (GPIO_NUM_13)   // B6
#define BSP_LCD_DATA4                   (GPIO_NUM_14)   // B7
#define BSP_LCD_DATA5                   (GPIO_NUM_21)   // G2
#define BSP_LCD_DATA6                   (GPIO_NUM_47)   // G3
#define BSP_LCD_DATA7                   (GPIO_NUM_48)   // G4
#define BSP_LCD_DATA8                   (GPIO_NUM_45)   // G5
#define BSP_LCD_DATA9                   (GPIO_NUM_38)   // G6
#define BSP_LCD_DATA10                  (GPIO_NUM_39)   // G7
#define BSP_LCD_DATA11                  (GPIO_NUM_40)   // R3
#define BSP_LCD_DATA12                  (GPIO_NUM_41)   // R4
#define BSP_LCD_DATA13                  (GPIO_NUM_42)   // R5
#define BSP_LCD_DATA14                  (GPIO_NUM_2)    // R6
#define BSP_LCD_DATA15                  (GPIO_NUM_1)    // R7
#define BSP_LCD_H_RES                   (480)
#define BSP_LCD_V_RES                   (480)
#define BSP_LCD_PIXEL_CLOCK_HZ          (18 * 1000 * 1000)
#define BSP_LCD_HSYNC_BACK_PORCH        (20)
#define BSP_LCD_HSYNC_FRONT_PORCH       (40)
#define BSP_LCD_HSYNC_PULSE_WIDTH       (13)
#define BSP_LCD_VSYNC_BACK_PORCH        (20)
#define BSP_LCD_VSYNC_FRONT_PORCH       (40)
#define BSP_LCD_VSYNC_PULSE_WIDTH       (15)
#define BSP_LCD_PCLK_ACTIVE_NEG         (false)
/******************* 请根据实际情况修改 *******************/

IRAM_ATTR static bool on_vsync_event(esp_lcd_panel_handle_t panel, const esp_lcd_rgb_panel_event_data_t *edata, void *user_ctx)
{
    BaseType_t need_yield = pdFALSE;

    // 此处执行需要的操作

    return (need_yield == pdTRUE);          // 返回 ture 表示需要 Freertos 调度器重新调度任务
}

esp_lcd_panel_handle_t bsp_lcd_init(void *arg)
{
    /* 如果采用 “3-line SPI + RGB” 需要先对屏幕进行初始化配置 */
#if BSP_IO_EXPANDER_EN
    ESP_ERROR_CHECK(arg);
    io = (esp_io_expander_handle_t)arg;
#endif
    BSP_ERROR_CHECK_RETURN_ERR(lcd_config());

    esp_lcd_panel_handle_t panel_handle = NULL;
    esp_lcd_rgb_panel_config_t panel_conf = {
        .clk_src = LCD_CLK_SRC_PLL160M,     // 设为 LCD_CLK_SRC_PLL160M 即可
        .psram_trans_align = 64,            // 设为 64 即可，若不设置，内部默认为 64
        .data_width = 16,                   // 数据线宽度，表示并行传输的位数（见 LCD 硬件介绍），仅支持如下两种:
                                            //    1. 若屏幕色彩类型为 16-bit RGB565，则设置为 16
                                            //    2. 若屏幕色彩类型为 8-bit RGB888，则设置为 8
        .disp_gpio_num = BSP_LCD_DISP       // 一般不使用该引脚，设为 -1 即可
        .de_gpio_num = BSP_LCD_DE,          // DE 信号引脚，在 LCD DE 模式（见 LCD 硬件详解）下使用，否则设为 -1
        .pclk_gpio_num = BSP_LCD_PCLK,      // PCLK 时钟引脚
        .vsync_gpio_num = BSP_LCD_VSYNC,    // VSYNC 帧同步信号引脚
        .hsync_gpio_num = BSP_LCD_HSYNC,    // HSYNC 行同步信号引脚
        .data_gpio_nums = {                 // D[15:0] 数据线，若为 8-bit RGB888，设置 D[7:0] 即可
            BSP_LCD_DATA0,
            BSP_LCD_DATA1,
            BSP_LCD_DATA2,
            BSP_LCD_DATA3,
            BSP_LCD_DATA4,
            BSP_LCD_DATA5,
            BSP_LCD_DATA6,
            BSP_LCD_DATA7,
            BSP_LCD_DATA8,
            BSP_LCD_DATA9,
            BSP_LCD_DATA10,
            BSP_LCD_DATA11,
            BSP_LCD_DATA12,
            BSP_LCD_DATA13,
            BSP_LCD_DATA14,
            BSP_LCD_DATA15,
        },
        .timings = {    // RGB 时序相关参数，需要根据屏厂给定的数据来设置
                        // 若没有，也可以查阅数据手册给定的范围大致设置一下，通常影响不大
            .pclk_hz = BSP_LCD_PIXEL_CLOCK_HZ,                  // PCLK 频率，一般大于数据手册给定的最小值，否则会闪白屏
            .h_res = BSP_LCD_H_RES,                             // 水平分辨率
            .v_res = BSP_LCD_V_RES,                             // 垂直分辨率
            .hsync_back_porch = BSP_LCD_HSYNC_BACK_PORCH,       // 水平后窗
            .hsync_front_porch = BSP_LCD_HSYNC_FRONT_PORCH,     // 水平前窗
            .hsync_pulse_width = BSP_LCD_HSYNC_PULSE_WIDTH,     // 水平脉宽
            .vsync_back_porch = BSP_LCD_VSYNC_BACK_PORCH,       // 垂直后窗
            .vsync_front_porch = BSP_LCD_VSYNC_FRONT_PORCH,     // 垂直前窗
            .vsync_pulse_width = BSP_LCD_VSYNC_PULSE_WIDTH,     // 垂直脉宽
            .flags.pclk_active_neg = BSP_LCD_PCLK_ACTIVE_NEG,   // true 表示 PCLK 下降沿有效，否则上升沿有效
        },
        .flags.fb_in_psram = 1,                                 // 设为 1 即可，由于 RGB 需要整帧 buffer 用于刷频，只能放在 PSRAM上
        // .flags.double_fb = 1,                                   // 1 表示使能内部双 buffer（整帧大小），否则仅单 buffer
        // .flags.refresh_on_demand = 1,                           // 0 表示内部自动刷新，即一帧传输完成后自动开始传输下一帧
                                                                // 1 表示开启手动刷新，即每帧都需要手动调用 esp_lcd_rgb_panel_refresh()
        // .bounce_buffer_size_px = 10 * BSP_LCD_H_RES,            // 设置 bounce buffer 的像素个数（内部会创建两个），见下面详述
    };
    ESP_ERROR_CHECK(esp_lcd_new_rgb_panel(&panel_conf, &panel_handle));
    esp_lcd_rgb_panel_event_callbacks_t cbs = {
        .on_vsync = on_vsync_event,
    };
    esp_lcd_rgb_panel_register_event_callbacks(panel_handle, &cbs, NULL);   // 注册回调函数，每传输完一帧会调用一次
    esp_lcd_panel_reset(panel_handle);
    esp_lcd_panel_init(panel_handle);
#if ESP_IDF_VERSION >= ESP_IDF_VERSION_VAL(5, 0, 0)
    ESP_ERROR_CHECK(esp_lcd_panel_disp_on_off(panel_handle, true));         // 通过 disp_gpio_num 引脚控制屏幕开启
#else
    ESP_ERROR_CHECK(esp_lcd_panel_disp_off(panel_handle, false));
#endif
    return panel_handle;
}
```

* **帧率**：分为接口帧率、渲染帧率和显示帧率

  1. **接口帧率**是指 RGB 接口向 LCD 驱动 IC 刷屏传输数据的帧率，决定了屏幕显示帧率的上限，计算公式如下：

    <div align=center >
    接口帧率 = pclk_hz /(h_res + h_back_porch + h_front_porch + h_pulse_width) * (v_res + v_back_porch + v_front_porch + v_pulse_width)
    </div>

  2. **渲染帧率**是指需要 CPU 计算渲染出动画效果（或图片编解码）的帧率，如 LVGL 运行动画时统计的 FPS，一般利用 LVGL Music Demo 统计的平均帧率来表征；

  3. **显示帧率**是指在屏幕上实际显示的动画效果的帧率，表示实际肉眼看到到的动画的流畅度，由接口帧率和渲染帧率共同决定，计算公式如下：

    <div align=center >
    显示帧率 = min(接口帧率, 渲染帧率)
    </div>

* **Bounce Buffer 机制**：驱动默认从 PSRAM 通过 DMA 传输数据到外设实现刷屏，而 Bounce buffer 通过指定大小的内部 SRAM，首先将数据从 PSRAM 通过 memcpy 搬运到内部 SRAM，然后通过 DMA 再传输至外设，以此来提升 PCLK 的设置上限，也能够避免操作 flash 引起的屏幕漂移问题，但是会提高 CPU 占用率，降低渲染帧率，详细讲解见[文档](https://docs.espressif.com/projects/esp-idf/en/latest/esp32s3/api-reference/peripherals/lcd.html#bounce-buffer-with-single-psram-frame-buffer)

* **刷屏说明**：与 8080/SPI LCD 驱动不同，RGB LCD 刷屏是从指定内存地址搬运刷屏数据到 PSRAM 内的帧 buffer，是采用 memcpy 的方式进行内存搬运，也就是说该函数调用完成就表示数据搬运完成，此时可以对原始内存区域进行修改（如让 LVGL 渲染计算），无需通过回调函数等待

* **防撕裂方案**：通过驱动内多 buffer 机制和 LVGL 的 buffering-mode，可以实现防撕裂方案，参考[例程](https://github.com/espressif/esp-dev-kits/tree/master/esp32-s3-lcd-ev-board/examples/lvgl_demos)

* **可用的 API**：后续可以利用 `panel_handle` 和 *[esp_lcd_panel_ops.h](https://github.com/espressif/esp-idf/blob/master/components/esp_lcd/include/esp_lcd_panel_ops.h)* 中的 API 来操作 LCD，如刷屏函数 `esp_lcd_panel_draw_bitmap()`。除此之外，还可以调用 *[esp_lcd_panel_rgb.h](https://github.com/espressif/esp-idf/blob/master/components/esp_lcd/include/esp_lcd_panel_rgb.h)* 中 RGB 特有的 API，如设置 PCLK 函数 `esp_lcd_rgb_panel_set_pclk()`

## 屏幕偏移问题

* 原因： 该问题通常是因为 RGB 需要连续数据输出，当执行写 Flash 操作或其他应用程序禁用或抢占 PSRAM 带宽时，数据传输将无法跟上 RGB 的时钟速度，从而导致永久的屏幕漂移。

* 解决方法： 如果应用不需要使用 Wi-Fi、BLE 和连续写 Flash 的操作，请先尝试[文档](https://docs.espressif.com/projects/espressif-esp-faq/zh_CN/latest/software-framework/peripherals/lcd.html#rgb-lcd)中的方法优化工程的配置并**尽量降低 PCLK**，否则，请使用 "PSRAM XIP" + "RGB Bounce buffer" 的方法，开启步骤如下：
    1. 确认 IDF 版本为较新（> 2022.12.12）的 release/v5.0 或 master，因为旧版本不支持 "PSRAM XIP" 的功能
    2. 确认 PSRAM 配置里面是否能开启  `SPIRAM_FETCH_INSTRUCTIONS` 和 `SPIRAM_RODATA` 这两项（如果 rodata 段数据过大，会导致 PSRAM 内存不够）
    3. 确认内存（SRAM）是否有余量，大概需要占用 10 * screen_width * 4 字节
    4. 需要将 Data cache line size 设置为 64 Byte（可设置 Data cache line size 为 32KB 以节省内存）
    5. 如以上均符合条件，那么就可以参考[文档](https://docs.espressif.com/projects/esp-idf/en/latest/esp32s3/api-reference/peripherals/lcd.html#bounce-buffer-with-single-psram-frame-buffer)修改 RGB 驱动（RGB 驱动里加上一行代码 .bounce_buffer_size_px = 10 * 屏幕宽度）
    6. 如操作 Wifi 仍存在屏幕漂移问题，可以尝试关闭 PSRAM 里 `SPIRAM_TRY_ALLOCATE_WIFI_LWIP` 一项（会占用较大 SRAM）

* 开启后带来的影响：
    1. CPU 使用率升高
    2. 可能会造成中断看门狗复位
    3. 会带来较大 SRAM 开销
