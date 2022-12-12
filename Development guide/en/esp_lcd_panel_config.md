# ESP_LCD driver adaptation

Currently, the official LCD driver ICs adapted based on the `esp_lcd` driver are as follows:

1. [esp_lcd](https://github.com/espressif/esp-idf/blob/7f4bcc36959b1c483897d643036f847eb08d270e/components/esp_lcd/include/esp_lcd_panel_vendor.h)：st7789、nt35510、ssd1306
2. [Package manager](https://components.espressif.com/)：gc9a01、ili9341、ili9488、ra8875、sh1107（continously updating）

**It should be noted that even if the driver IC is the same, different screens often require different register configuration parameters, and screen manufacturers usually provide matching configuration parameters (codes), so it is recommended to use the above two methods to obtain codes of similar driver ICs, according to Modify the actual parameters of your own screen.**

## Driver template

This section provides a set of common driver codes for common **SPI/8080** interface LCD driver ICs (ili9341, st7789, gc9a01, etc.), users can adjust them according to the actual configuration parameters of the screen, and then use `esp_lcd` to easily Adapt to your own screen.

Take st7789 as an example, **The detailed adaptation process will be explained later**:

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

## Framework

### Implementation principle

`esp_lcd` internally provides a set of standard LCD operation APIs (including initialization, reset, refresh, etc.), which are located in the *esp_lcd_panel_interface.h* header file, and the way for users to adapt the driver is to implement these interfaces and Use the `esp_lcd_panel_handle_t` variable to associate the implementation with the interface. Through this method, users can uniformly use the API in *esp_lcd_panel_ops.h* to operate LCDs with different interfaces (SPI, RGB, etc.), and the corresponding relationship of the interfaces is as follows:

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

When the user replaces a screen with a different interface, it only needs to replace a set of drivers without modifying the code of the application layer.

### Adaptation steps

1. **Confirm Compatibility**: The simpler method is to check the data sheet of the screen driver IC to confirm the screen refresh process (command, timing, etc.), and check whether it is consistent by comparing the `panel_st7789_draw_bitmap()` function in the template, otherwise Please refer to other examples to modify or adapt by yourself
2. **Name Replacement**: Use an editor to search the keyword "st7789" in the template and globally replace it with the target IC name (such as st7701)
3. **Modify register configuration**: Only the `vendor_specific_init` array and the `panel_st7789_init()` function need to be modified in the entire template. Among them, the former needs to be modified according to the configuration parameters given by the screen manufacturer, and the `LCD_CONFIG_DATA_LEN_MAX` macro should be modified according to the maximum byte length in the data; if the command has delay or special command requirements, the latter can be adjusted by itself.

## Example of use

```
    // Users need to initialize the io_handle variable in advance according to the interface type

    esp_lcd_panel_handle_t panel_handle = NULL;
    esp_lcd_panel_dev_config_t panel_config = {
        .reset_gpio_num = PIN_NUM_LCD_RST,          // 复位引脚，没有则设为 -1
        .color_space = ESP_LCD_COLOR_SPACE_RGB,     // LCD 的 RGB 顺序，一般默认为 ESP_LCD_COLOR_SPACE_RGB
                                                    // 内部通过 0x36 命令进行实现
        .bits_per_pixel = 16,                       // RGB565-16、RGB666-18
    };
    ESP_ERROR_CHECK(lcd_new_panel_st7789(io_handle, &panel_config, &panel_handle));
    ESP_ERROR_CHECK(esp_lcd_panel_reset(panel_handle));     // 复位，若设置了复位引脚，则硬件复位，否则软件复位
    ESP_ERROR_CHECK(esp_lcd_panel_init(panel_handle));
#if ESP_IDF_VERSION >= ESP_IDF_VERSION_VAL(5, 0, 0)
    ESP_ERROR_CHECK(esp_lcd_panel_disp_on_off(panel_handle, true));
#else
    ESP_ERROR_CHECK(esp_lcd_panel_disp_off(panel_handle, false));
#endif
```

* **Available API**: You can use the API in `panel_handle` and *esp_lcd_panel_ops.h* to operate the LCD later, such as the refresh function `esp_lcd_panel_draw_bitmap()`

* **`esp_lcd_panel_draw_bitmap()`**: 8080/SPI LCD refresh screen transfers data from the specified memory address to the peripheral, usually using DMA, which means that the data is still being processed by DMA after the function call is completed Transmission, the memory area in use cannot be modified at this time (such as LVGL rendering calculation), so it is necessary to judge whether the transmission is completed through the callback function registered when the bus is initialized
