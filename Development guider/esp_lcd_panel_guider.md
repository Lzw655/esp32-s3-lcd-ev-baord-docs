# 配置 esp_lcd_panel（LCD 寄存器）

目前官方基于 `esp_lcd` 驱动适配过的 LCD 驱动 IC 如下：

1. [esp_lcd](https://github.com/espressif/esp-idf/blob/7f4bcc36959b1c483897d643036f847eb08d270e/components/esp_lcd/include/esp_lcd_panel_vendor.h)：st7789、nt35510、ssd1306
2. [包管理器](https://components.espressif.com/)：gc9a01、ili9341、ili9488、ra8875、sh1107（持续更新中）

**需注意，即使驱动 IC 相同，不同的屏幕往往需要不同的寄存器配置参数，而且屏幕厂商通常会给配套的配置参数（代码），因此推荐利用上面两种途径获取相似驱动 IC 的代码，根据自己屏幕的实际参数进行修改。**

## 驱动模板

本节提供了一套针对常见 **SPI/8080** 接口 LCD 驱动 IC（ili9341、st7789、gc9a01 等型号）通用的驱动代码，用户按照屏幕实际的配置参数进行调整，即可使用 `esp_lcd` 轻松适配自己的屏幕。

以 st7789 为例，后续会讲解详细适配过程：

* **lcd_panel_st7789.h**:
```
/*
 * SPDX-FileCopyrightText: 2021 Espressif Systems (Shanghai) CO LTD
 *
 * SPDX-License-Identifier: Apache-2.0
 */

#pragma once

#include "esp_lcd_panel_vendor.h"

#ifdef __cplusplus
extern "C" {
#endif

/**
 * @brief Create LCD panel
 *
 * @param[in] io: LCD panel IO handle
 * @param[in] panel_dev_config: General panel device configuration
 * @param[out] ret_panel: Returned LCD panel handle
 * @return
 *          - ESP_OK: on success, otherwise fail
 */
esp_err_t lcd_new_panel_st7789(const esp_lcd_panel_io_handle_t io, const esp_lcd_panel_dev_config_t *panel_dev_config, esp_lcd_panel_handle_t *ret_panel);

#ifdef __cplusplus
}
#endif
```
* **lcd_panel_st7789.c**:

```
/*
 * SPDX-FileCopyrightText: 2021-2022 Espressif Systems (Shanghai) CO LTD
 *
 * SPDX-License-Identifier: Apache-2.0
 */

#include <stdlib.h>
#include <sys/cdefs.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "driver/gpio.h"
#include "esp_check.h"
#include "esp_lcd_panel_commands.h"
#include "esp_lcd_panel_interface.h"
#include "esp_lcd_panel_io.h"
#include "esp_lcd_panel_ops.h"
#include "esp_lcd_panel_vendor.h"
#include "esp_log.h"

#include "sdkconfig.h"

/******************* 请根据实际情况修改 *******************/
#define LCD_CONFIG_DATA_LEN_MAX         (16)
/******************* 请根据实际情况修改 *******************/

static const char *TAG = "lcd_panel_st7789";

static esp_err_t panel_st7789_del(esp_lcd_panel_t *panel);
static esp_err_t panel_st7789_reset(esp_lcd_panel_t *panel);
static esp_err_t panel_st7789_init(esp_lcd_panel_t *panel);
static esp_err_t panel_st7789_draw_bitmap(esp_lcd_panel_t *panel, int x_start, int y_start, int x_end, int y_end, const void *color_data);
static esp_err_t panel_st7789_invert_color(esp_lcd_panel_t *panel, bool invert_color_data);
static esp_err_t panel_st7789_mirror(esp_lcd_panel_t *panel, bool mirror_x, bool mirror_y);
static esp_err_t panel_st7789_swap_xy(esp_lcd_panel_t *panel, bool swap_axes);
static esp_err_t panel_st7789_set_gap(esp_lcd_panel_t *panel, int x_gap, int y_gap);
#if ESP_IDF_VERSION >= ESP_IDF_VERSION_VAL(5, 0, 0)
static esp_err_t panel_st7789_disp_on_off(esp_lcd_panel_t *panel, bool on_off);
#else
static esp_err_t panel_st7789_disp_off(esp_lcd_panel_t *panel, bool off);
#endif

typedef struct {
    esp_lcd_panel_t base;
    esp_lcd_panel_io_handle_t io;
    int reset_gpio_num;
    bool reset_level;
    int x_gap;
    int y_gap;
    unsigned int bits_per_pixel;
    uint8_t madctl_val; // save current value of LCD_CMD_MADCTL register
    uint8_t colmod_cal; // save surrent value of LCD_CMD_COLMOD register
} st7789_panel_t;

esp_err_t lcd_new_panel_st7789(const esp_lcd_panel_io_handle_t io, const esp_lcd_panel_dev_config_t *panel_dev_config, esp_lcd_panel_handle_t *ret_panel)
{
    esp_err_t ret = ESP_OK;
    st7789_panel_t *st7789 = NULL;
    ESP_GOTO_ON_FALSE(io && panel_dev_config && ret_panel, ESP_ERR_INVALID_ARG, err, TAG, "invalid argument");
    st7789 = calloc(1, sizeof(st7789_panel_t));
    ESP_GOTO_ON_FALSE(st7789, ESP_ERR_NO_MEM, err, TAG, "no mem for st7789 panel");

    if (panel_dev_config->reset_gpio_num >= 0) {
        gpio_config_t io_conf = {
            .mode = GPIO_MODE_OUTPUT,
            .pin_bit_mask = 1ULL << panel_dev_config->reset_gpio_num,
        };
        ESP_GOTO_ON_ERROR(gpio_config(&io_conf), err, TAG, "configure GPIO for RST line failed");
    }

    switch (panel_dev_config->color_space) {
    case ESP_LCD_COLOR_SPACE_RGB:
        st7789->madctl_val = 0;
        break;
    case ESP_LCD_COLOR_SPACE_BGR:
        st7789->madctl_val |= LCD_CMD_BGR_BIT;
        break;
    default:
        ESP_GOTO_ON_FALSE(false, ESP_ERR_NOT_SUPPORTED, err, TAG, "unsupported color space");
        break;
    }

    switch (panel_dev_config->bits_per_pixel) {
    case 16:
        st7789->colmod_cal = 0x55;
        break;
    case 18:
        st7789->colmod_cal = 0x66;
        break;
    default:
        ESP_GOTO_ON_FALSE(false, ESP_ERR_NOT_SUPPORTED, err, TAG, "unsupported pixel width");
        break;
    }

    st7789->io = io;
    st7789->bits_per_pixel = panel_dev_config->bits_per_pixel;
    st7789->reset_gpio_num = panel_dev_config->reset_gpio_num;
    st7789->reset_level = panel_dev_config->flags.reset_active_high;
    st7789->base.del = panel_st7789_del;
    st7789->base.reset = panel_st7789_reset;
    st7789->base.init = panel_st7789_init;
    st7789->base.draw_bitmap = panel_st7789_draw_bitmap;
    st7789->base.invert_color = panel_st7789_invert_color;
    st7789->base.set_gap = panel_st7789_set_gap;
    st7789->base.mirror = panel_st7789_mirror;
    st7789->base.swap_xy = panel_st7789_swap_xy;
#if ESP_IDF_VERSION >= ESP_IDF_VERSION_VAL(5, 0, 0)
    st7789->base.disp_on_off = panel_st7789_disp_on_off;
#else
    st7789->base.disp_off = panel_st7789_disp_off;
#endif
    *ret_panel = &(st7789->base);
    ESP_LOGD(TAG, "new st7789 panel @%p", st7789);

    return ESP_OK;

err:
    if (st7789) {
        if (panel_dev_config->reset_gpio_num >= 0) {
            gpio_reset_pin(panel_dev_config->reset_gpio_num);
        }
        free(st7789);
    }
    return ret;
}

static esp_err_t panel_st7789_del(esp_lcd_panel_t *panel)
{
    st7789_panel_t *st7789 = __containerof(panel, st7789_panel_t, base);

    if (st7789->reset_gpio_num >= 0) {
        gpio_reset_pin(st7789->reset_gpio_num);
    }
    ESP_LOGD(TAG, "del st7789 panel @%p", st7789);
    free(st7789);
    return ESP_OK;
}

static esp_err_t panel_st7789_reset(esp_lcd_panel_t *panel)
{
    st7789_panel_t *st7789 = __containerof(panel, st7789_panel_t, base);
    esp_lcd_panel_io_handle_t io = st7789->io;

    // perform hardware reset
    if (st7789->reset_gpio_num >= 0) {
        gpio_set_level(st7789->reset_gpio_num, st7789->reset_level);
        vTaskDelay(pdMS_TO_TICKS(10));
        gpio_set_level(st7789->reset_gpio_num, !st7789->reset_level);
        vTaskDelay(pdMS_TO_TICKS(10));
    } else { // perform software reset
        esp_lcd_panel_io_tx_param(io, LCD_CMD_SWRESET, NULL, 0);
        vTaskDelay(pdMS_TO_TICKS(20)); // spec, wait at least 5m before sending new command
    }

    return ESP_OK;
}

typedef struct {
    uint8_t cmd;
    uint8_t data[LCD_CONFIG_DATA_LEN_MAX];
    uint8_t data_bytes; // Length of data in above data array; 0xFF = end of cmds.
} lcd_init_cmd_t;

static const lcd_init_cmd_t vendor_specific_init[] = {
/******************* 请根据实际情况修改 *******************/
    {0x36, {0x00}, 1},
    {0x3a, {0x55}, 1},
    {0xb2, {0x0c, 0x0c, 0x00, 0x33, 0x33}, 5},
    {0xb7, {0x35}, 1},
    {0xbb, {0x35}, 1},
    {0xc0, {0x2c}, 1},
    {0xc2, {0x01}, 1},
    {0xc3, {0x0b}, 1},
    {0xc4, {0x20}, 1},
    {0xc6, {0x0f}, 1},
    {0xd0, {0xa4, 0xa1}, 2},
    {0xe0, {0xd0, 0x00, 0x02, 0x07, 0x0b, 0x1a, 0x31, 0x54, 0x40, 0x29, 0x12, 0x12, 0x12, 0x17}, 14},
    {0xe1, {0xd0, 0x00, 0x02, 0x07, 0x05, 0x25, 0x2d, 0x44, 0x45, 0x1c, 0x18, 0x16, 0x1c, 0x1d}, 14},
/******************* 请根据实际情况修改 *******************/

    {0x00, {0x00}, 0xff},
};

static esp_err_t panel_st7789_init(esp_lcd_panel_t *panel)
{
    st7789_panel_t *st7789 = __containerof(panel, st7789_panel_t, base);
    esp_lcd_panel_io_handle_t io = st7789->io;
    // LCD goes into sleep mode and display will be turned off after power on reset, exit sleep mode first
    esp_lcd_panel_io_tx_param(io, LCD_CMD_SLPOUT, NULL, 0);
    vTaskDelay(pdMS_TO_TICKS(120));
    for (uint8_t i = 0; vendor_specific_init[i].data_bytes != 0xff; i++) {
        esp_lcd_panel_io_tx_param(io, vendor_specific_init[i].cmd, vendor_specific_init[i].data, vendor_specific_init[i].data_bytes & 0x1F);
    }
    esp_lcd_panel_io_tx_param(io, LCD_CMD_COLMOD, (uint8_t []){st7789->colmod_cal}, 1);
    esp_lcd_panel_io_tx_param(io, LCD_CMD_MADCTL, (uint8_t []){st7789->madctl_val}, 1);

    return ESP_OK;
}

static esp_err_t panel_st7789_draw_bitmap(esp_lcd_panel_t *panel, int x_start, int y_start, int x_end, int y_end, const void *color_data)
{
    st7789_panel_t *st7789 = __containerof(panel, st7789_panel_t, base);
    assert((x_start < x_end) && (y_start < y_end) && "start position must be smaller than end position");
    esp_lcd_panel_io_handle_t io = st7789->io;

    x_start += st7789->x_gap;
    x_end += st7789->x_gap;
    y_start += st7789->y_gap;
    y_end += st7789->y_gap;

    // define an area of frame memory where MCU can access
    esp_lcd_panel_io_tx_param(io, LCD_CMD_CASET, (uint8_t[]) {
        (x_start >> 8) & 0xFF,
        x_start & 0xFF,
        ((x_end - 1) >> 8) & 0xFF,
        (x_end - 1) & 0xFF,
    }, 4);
    esp_lcd_panel_io_tx_param(io, LCD_CMD_RASET, (uint8_t[]) {
        (y_start >> 8) & 0xFF,
        y_start & 0xFF,
        ((y_end - 1) >> 8) & 0xFF,
        (y_end - 1) & 0xFF,
    }, 4);
    // transfer frame buffer
    size_t len = (x_end - x_start) * (y_end - y_start) * st7789->bits_per_pixel / 8;
    esp_lcd_panel_io_tx_color(io, LCD_CMD_RAMWR, color_data, len);

    return ESP_OK;
}

static esp_err_t panel_st7789_invert_color(esp_lcd_panel_t *panel, bool invert_color_data)
{
    st7789_panel_t *st7789 = __containerof(panel, st7789_panel_t, base);
    esp_lcd_panel_io_handle_t io = st7789->io;
    int command = 0;
    if (invert_color_data) {
        command = LCD_CMD_INVON;
    } else {
        command = LCD_CMD_INVOFF;
    }
    esp_lcd_panel_io_tx_param(io, command, NULL, 0);
    return ESP_OK;
}

static esp_err_t panel_st7789_mirror(esp_lcd_panel_t *panel, bool mirror_x, bool mirror_y)
{
    st7789_panel_t *st7789 = __containerof(panel, st7789_panel_t, base);
    esp_lcd_panel_io_handle_t io = st7789->io;
    if (mirror_x) {
        st7789->madctl_val |= LCD_CMD_MX_BIT;
    } else {
        st7789->madctl_val &= ~LCD_CMD_MX_BIT;
    }
    if (mirror_y) {
        st7789->madctl_val |= LCD_CMD_MY_BIT;
    } else {
        st7789->madctl_val &= ~LCD_CMD_MY_BIT;
    }
    esp_lcd_panel_io_tx_param(io, LCD_CMD_MADCTL, (uint8_t[]) {
        st7789->madctl_val
    }, 1);
    return ESP_OK;
}

static esp_err_t panel_st7789_swap_xy(esp_lcd_panel_t *panel, bool swap_axes)
{
    st7789_panel_t *st7789 = __containerof(panel, st7789_panel_t, base);
    esp_lcd_panel_io_handle_t io = st7789->io;
    if (swap_axes) {
        st7789->madctl_val |= LCD_CMD_MV_BIT;
    } else {
        st7789->madctl_val &= ~LCD_CMD_MV_BIT;
    }
    esp_lcd_panel_io_tx_param(io, LCD_CMD_MADCTL, (uint8_t[]) {
        st7789->madctl_val
    }, 1);
    return ESP_OK;
}

static esp_err_t panel_st7789_set_gap(esp_lcd_panel_t *panel, int x_gap, int y_gap)
{
    st7789_panel_t *st7789 = __containerof(panel, st7789_panel_t, base);
    st7789->x_gap = x_gap;
    st7789->y_gap = y_gap;
    return ESP_OK;
}

#if ESP_IDF_VERSION >= ESP_IDF_VERSION_VAL(5, 0, 0)
static esp_err_t panel_st7789_disp_on_off(esp_lcd_panel_t *panel, bool on_off)
{
    st7789_panel_t *st7789 = __containerof(panel, st7789_panel_t, base);
    esp_lcd_panel_io_handle_t io = st7789->io;
    int command = 0;
    if (on_off) {
        command = LCD_CMD_DISPON;
    } else {
        command = LCD_CMD_DISPOFF;
    }
    esp_lcd_panel_io_tx_param(io, command, NULL, 0);
    return ESP_OK;
}
#else
static esp_err_t panel_st7789_disp_off(esp_lcd_panel_t *panel, bool off)
{
    st7789_panel_t *st7789 = __containerof(panel, st7789_panel_t, base);
    esp_lcd_panel_io_handle_t io = st7789->io;
    int command = 0;
    if (off) {
        command = LCD_CMD_DISPOFF;
    } else {
        command = LCD_CMD_DISPON;
    }
    esp_lcd_panel_io_tx_param(io, command, NULL, 0);
    return ESP_OK;
}
#endif
```

## 模板解析

### 实现原理

`esp_lcd` 内部提供了一套标准的 LCD 操作 API（包含了初始化、复位、刷屏等），位于 *esp_lcd_panel_interface.h* 头文件中，而用户适配驱动的方式就是要去实现这些接口，并利用 `esp_lcd_panel_handle_t` 变量将实现与接口进行关联。通过这样的方法，用户可以统一使用 *esp_lcd_panel_ops.h* 中的 API 对不同接口（SPI、RGB 等）的 LCD 进行操作，接口的对应关系如下：

|     esp_lcd_panel_ops.h      | esp_lcd_panel_interface.h |
| ---------------------------- | ------------------------- |
| esp_lcd_panel_reset()        | reset()                   |
| esp_lcd_panel_init()         | init()                    |
| esp_lcd_panel_del()          | del()                     |
| esp_lcd_panel_draw_bitmap()  | draw_bitmap()             |
| esp_lcd_panel_mirror()       | mirror()                  |
| esp_lcd_panel_swap_xy()      | swap_xy()                 |
| esp_lcd_panel_set_gap()      | set_gap()                 |
| esp_lcd_panel_invert_color() | invert_color()            |
| esp_lcd_panel_disp_on_off()  | disp_on_off()             |

当用户更换不同接口的屏幕后，仅需更换一套驱动，而无需修改应用层的代码。

### 适配步骤

1. **确认兼容性**：比较简单的方法是，查看屏幕驱动 IC 的数据手册确认刷屏过程（命令、时序等），通过对比模板中的 `panel_st7789_draw_bitmap()` 函数查看是否一致，否则请参照其他示例修改或自行进行适配
2. **名称替换**：用编辑器在模板中搜索关键词 “st7789” 并全局替换为目标 IC 名称（如 st7701）
3. **修改寄存器配置**：整个模板中仅需修改 `vendor_specific_init` 数组 和 `panel_st7789_init()` 函数。其中，前者需要根据屏幕厂商给的配置参数进行修改，并根据数据中的最大字节长度修改 `LCD_CONFIG_DATA_LEN_MAX` 宏；如果命令有延时或特殊命令等要求，可自行对后者进行调整。
