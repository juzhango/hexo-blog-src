---
title: uis7870sc点亮lcd
abbrlink: 971ea0e1
date: 2024-03-27 09:17:36
tags: [展锐]
---


# LK 阶段

配置编译选项，这些宏用于配置是否需要编译的`lcd`驱动文件。

```diff
Index: project/uis7870sc_2h10.mk
===================================================================
--- project/uis7870sc_2h10.mk   (revision 2)
+++ project/uis7870sc_2h10.mk   (working copy)
@@ -100,6 +100,7 @@
 # attach lcd
 LCD_G40396_TRULY_MIPI_FHD := 1
 LCD_NT36672E_TRULY_MIPI_FHD := 1
+LCD_SC7705_TRULY_MIPI_FHD :1

 # lcd mipi clk config
 GLOBAL_DEFINES += \
Index: target/uis7870sc_2h10/init.c
```

配置宏开关，添加`lcd`初始化代码路径。

```diff
Index: platform/sprd_shared/driver/video/sprd/rules.mk
===================================================================
--- platform/sprd_shared/driver/video/sprd/rules.mk     (revision 2)
+++ platform/sprd_shared/driver/video/sprd/rules.mk     (working copy)
@@ -174,6 +174,12 @@
        $(LOCAL_DIR)/lcd/lcd_hx8394f_mipi.c
 endif

+ifeq ($(LCD_SC7705_TRULY_MIPI_FHD),1)
+GLOBAL_DEFINES += CONFIG_LCD_SC7705_TRULY_MIPI_FHD
+MODULE_SRCS += \
+       $(LOCAL_DIR)/lcd/lcd_sc7705_truly_mipi_fhd.c
+endif
+
 ifeq ($(PLATFORM), qogirl6)
 MODULE_SRCS += \
     $(LOCAL_DIR)/dphy/pll/megacores_sharkl5.c \

```

添加 `lcd` 驱动，驱动通过 `sc7705_readid()` 来确定 `lcd_name`。

```shell
$ svn status
?       platform/sprd_shared/driver/video/sprd/lcd/lcd_sc7705_truly_mipi_fhd.c
```

`lcd_sc7705_truly_mipi_fhd.c`
```c
static uint8_t init_data[] = {
};

static int mipi_dsi_send_cmds(struct sprd_dsi *dsi, void *data)
{
}

static struct panel_ops xxx_ops = {
	.init = xxx_init,
	.read_id = xxx_readid,
	.power = xxx_power,
};

static struct panel_info xxx_info = {
};

struct panel_driver xxx_xxx_driver = {
	.info = &xxx_info,
	.ops = &xxx_ops,
};
```

将驱动注册到 `bootloader`

```diff
Index: platform/sprd_shared/driver/video/sprd/lcd/panel_cfg.h
===================================================================
--- platform/sprd_shared/driver/video/sprd/lcd/panel_cfg.h      (revision 2)
+++ platform/sprd_shared/driver/video/sprd/lcd/panel_cfg.h      (working copy)
@@ -62,8 +62,15 @@
 extern struct panel_driver td4160_boe_driver;
 extern struct panel_driver lcd_hx8394f_mipi_driver;
 extern struct panel_driver ili9881d_truly_driver;
+extern struct panel_driver sc7705_truly_driver;

 static struct panel_cfg supported_panel[] = {
+#ifdef CONFIG_LCD_SC7705_TRULY_MIPI_FHD
+       {
+               .lcd_id = 0x7705,
+               .drv = &sc7705_truly_driver,
+       },
+#endif
 #ifdef CONFIG_FB_LCD_HX8394F_MIPI
        {
                .lcd_id = 0x8394,

```

`bootloader` 会通过 `panel_cfg.h ` 遍历所有驱动。

`platform/sprd_shared/driver/video/sprd/sprd_panel.c`
```c
int sprd_panel_probe(void)
{
	for (i = 0; i < ARRAY_SIZE(supported_panel); i++) {
		panel_drv = supported_panel[i].drv;
		info = panel_drv->info;
		ops = panel_drv->ops;
		...
}
```

---
# Kernel

lcd 先在 bootloader 阶段点亮后，会将 `lcd_name` 传递到 `bootargs`，例如：`lcd_name=lcd_sc7705_truly_mipi_fhd`，**这个 `lcd_name` 要与设备树的 `lcd_xxx.dts` 节点名一致。**

需要配置的 GPIO：

- sprd,backlight
- avdd-gpio
- avee-gpio
- reset-gpio

```c
#include "lcd/lcd_sc7705_truly_mipi_fhd.dtsi"

&dsi {
	status = "okay";
	#address-cells = <1>;
	#size-cells = <0>;

	panel: panel {
		compatible = "sprd,generic-mipi-panel";
		#address-cells = <1>;
		#size-cells = <0>;
		reg = <0>;
		sprd,backlight = <&pwm_backlight>;
		avdd-gpio = <&ap_gpio 192 GPIO_ACTIVE_HIGH>;
		avee-gpio = <&ap_gpio 191 GPIO_ACTIVE_HIGH>;
		reset-gpio = <&ap_gpio 11 GPIO_ACTIVE_HIGH>;
		port@1 {
			reg = <1>;
			panel_in: endpoint {
				remote-endpoint = <&dphy_out>;
			};
		};
	};
};
```

`lcd/lcd_sc7705_truly_mipi_fhd.dtsi`

```c
/ {
	lcds {
		lcd_sc7705_truly_mipi_fhd: lcd_sc7705_truly_mipi_fhd {

			sprd,dsi-work-mode = <1>; /* video burst mode*/
			sprd,dsi-lane-number = <4>;
			sprd,dsi-color-format = "rgb888";

			sprd,phy-bit-clock = <455000>;	/* kbps */
			sprd,phy-escape-clock = <20000>; /* kHz */

			sprd,reset-on-sequence = <1 15>, <0 15>, <1 15>;
			sprd,reset-off-sequence = <0 5>;

			sprd,use-dcs-write;

			sprd,initial-command = [
			];
			sprd,sleep-in-command = [
			];
			sprd,sleep-out-command = [
			];

			display-timings {
				native-mode = <&sc7705truly_timing0>;
				sc7705truly_timing0: timing0 {
					clock-frequency = <68250000>;
					hactive = <720>;
					vactive = <1280>;
					hback-porch = <40>;
					hfront-porch = <40>;
					vback-porch = <14>;
					vfront-porch = <18>;
					hsync-len = <20>;
					vsync-len = <3>;
				};
			};
		};
	};
};
```

---

# 时钟说明

## pixel_clk

`pixel_clk(DPI CLK)` 的单位为 `pixel/s`

```
DPI CLK = (HActive + HSYNC + HBP + HFP) * (VActive + VSYNC + VBP + VFP) * FPS
```

受限于硬件配置，DPI CLK 只有几个固定的 CLK SOURCE，因此只能以这几个 CLK SOURCE 及其分频得到的数值为基准，调整 porch 参数，来确定 DPI CLK。

![3.jpg](https://juzhango-1257120269.cos.ap-shanghai.myqcloud.com/240304/3.jpg)

假设有两个相邻的取值为 a 和 b，a 小于 b，且 a 与 b 相差较大，可能存在如下情况：DPI CLK 设置为 a，不满足传输速率要求；DPI CLK 设置为 b，导致 MIPI CLK 过大，超出了屏幕的硬件限制（屏幕硬件有针对 MIPI CLK 的最大传输速率限制，超出限制会显示异常），此时无法正常显示。

于是引入了六分频的配置方案，打开六分频后，DPI CLK 的 CLK SOURCE 为 MIPI CLK，且 DPI CLK 就是由 MIPI CLK 六分频得到。由于 MIPI CLK 可在一个区间内任意配置，且打开六分频后，`DPI CLK =  MIPI CLK/6`，这样 DPI CLK 也可在一个区间内任意配置，只要满足传输速率要求即可，配置更加灵活，从而支持更多的屏。

## phy_freq

`DPI CLK(pixel_clk)`为像素时钟，单位为`pixel/s`，其中`pixel`为3Byte，故单位也为`24bit/s`。`MIPI CLK`单位为`bps`，1 bps = 1bit/s（单Lane）。

DPU 将像素数据按照 DPI CLK 的速率发送给 DSI，DSI 将这些数据按照 MIPI CLK 速率发送给屏端，前后数据量相同，故 DPI CLK 与 MIPI CLK 需要满足如下关系：
```
DPI CLK * 3 * 8 = MIPI CLK * 4
```

MIPI 每发送一行数据，都会进入低功耗模式，切换低功耗模式的过程有时间消耗，所以MIPI CLK 会配置的大一些，可按照如下规则配置：
```
MIPI CLK * 0.9 = DPI CLK * 6
```

当打开六分频后，DPI CLK 与 MIPI CLK 同源，MIPI 不需要进入低功耗模式，省去了切换低功耗模式的时间，此时 MIPI CLK 与 DPI CLK 满足 6 倍关系：
```
MIPI CLK = DPI CLK * 6
```

**注意：**

- 开启 6 分频功能除了修改 DPI CLK 和 MIPI CLK 外，还需要加上如下配置：
	- `dpi_clk_div = 6`
- 使用 6 分频功能时，`htotal(hbp+hfp+hsync+width)` 必须是 4 的倍数，`pixel_clk` 需要被 6000 整除。  
- 使用 6 分频功能同时要配置如下两项：  
	- `video_lp_cmd_enable = true`
	- `hporch_lp_disable = true`

---

# 初始化命令

遵循DSI协议报文格式

```c
struct dsi_cmd_desc {
	uint8_t data_type;
	uint8_t wait; // 延时时间(ms)
	uint8_t wc_h;
	uint8_t wc_l;
	uint8_t payload[];
}
```

## 转换格式

供应商一般给的是伪代码需要转换
```c
SSD_SEND(0x01,0x00,0x00);
SSD_SEND(0x01,0xFF,0x82,0x01,0x01);
SSD_SEND(0x01,0x00,0x80);
SSD_SEND(0x01,0xFF,0x82,0x01);
SSD_SEND(0x01,0x00,0x93);
SSD_SEND(0x01,0xC5,0x70);
SSD_SEND(0x01,0x00,0x97);
SSD_SEND(0x01,0xC5,0x70);
SSD_SEND(0x01,0x00,0x9E);
SSD_SEND(0x01,0xC5,0x0A);
SSD_SEND(0x01,0x00,0x9A);
```

![1.png](https://juzhango-1257120269.cos.ap-shanghai.myqcloud.com/240314/1.png)

通过chatgpt生成代码：

```c
// 引入必要的头文件
#include <stdio.h>  // 提供printf、scanf等输入输出功能
#include <string.h> // 提供字符串处理函数如strstr、strtok和strcat等
#include <stdlib.h> // 提供strtol等转换函数

// 定义原始待处理的字符串文本
const char *text =
"SSD_SEND(0x01,0x00,0x00);"
"SSD_SEND(0x01,0xFF,0x82,0x01,0x01);"
"SSD_SEND(0x01,0x00,0x80);"
"SSD_SEND(0x01,0xFF,0x82,0x01);"
"SSD_SEND(0x01,0x00,0x93);"
"SSD_SEND(0x01,0xC5,0x70);"
"SSD_SEND(0x01,0x00,0x97);"
"SSD_SEND(0x01,0xC5,0x70);"
"SSD_SEND(0x01,0x00,0x9E);"
"SSD_SEND(0x01,0xC5,0x0A);"
"SSD_SEND(0x01,0x00,0x9A);"
;

// 函数：convert，用于解析并转换输入的字符串
void convert(const char *input)
{
    const char *ptr = input;
    int arg_count = 0;

    // 查找字符串中的"SSD_SEND("子串
    ptr = strstr(input, "SSD_SEND(");

    if (ptr != NULL)
    {
        // 移动指针至"SSD_SEND("之后的位置
        ptr += 8;

        // 打印初始部分（39 00）
        printf("39 00 ");

        // 统计参数个数
        while (*ptr != '\0' && *ptr != ')')
        {
            if (*ptr == '0' && *(ptr + 1) == 'x')
            {
                // 跳过"0x"及其后的十六进制数字
                ptr += 2;
                while ((*ptr >= '0' && *ptr <= '9') || (*ptr >= 'A' && *ptr <= 'F'))
                {
                    ptr++;
                }
                arg_count++;
            }
            ptr++;
        }

        // 将参数个数减1后以十六进制字符打印
        printf("%02X", arg_count - 1);

        // 打印一个空格
        printf(" ");
    }
    else
    {
        printf("SSD_SEND not found in input.");
        return;
    }

    // 解析并打印参数
    ptr = strstr(input, "0x");
    int first = 1;
    while (ptr != NULL)
    {
        if (!first)
        {
            // 将十六进制字符串转换为整型
            int value = (int)strtol(ptr, (char **)&ptr, 16);
            // 将值以十六进制字符打印
            printf("%02X ", value);
        }
        else
        {
            first = 0;
            // 跳过"0x"及其后的十六进制数字
            ptr += 2;
            while ((*ptr >= '0' && *ptr <= '9') || (*ptr >= 'A' && *ptr <= 'F'))
            {
                ptr++;
            }
        }
        // 查找下一个"0x"出现的位置
        ptr = strstr(ptr, "0x");
    }

    printf("\n");
}

// 函数：splitText，按分号将文本分割成多行
void splitText(const char *text, char *line)
{
    const char *delimiter = ";"; // 使用分号作为分隔符
    const char *ptr = text;

    while (*ptr != '\0')
    {
        if (*ptr == ';')
        {
            strcat(line, ";");  // 追加分号
            strcat(line, "\n"); // 追加换行符
        }
        else
        {
            strncat(line, ptr, 1);
        }
        ptr++;
    }
}

// 主函数main
int main()
{
    char all[50000] = ""; // 初始化存储分割后文本的数组

    // 将原始文本分割并存入all数组中
    splitText(text, all);

    // printf("%s", all); // 输出分割后的文本（可选）

    const char *delimiter = "\n"; // 使用换行符作为分隔符
    char *line = strtok(all, delimiter); // 分割第一行

    // 遍历所有分割出来的行并进行转换处理
    while (line != NULL)
    {
        convert(line); // 对每一行调用convert函数进行处理
        line = strtok(NULL, delimiter); // 获取下一行
    }

    return 0; // 程序正常结束
}
```

---

# 其他配置项

- **lcd_name**: LCD 节点名，必须和 Kernel 中的 LCD DTS 节点名称保持一致，否则 Kernel  无法匹配到正确的 LCD 驱动。
- **nc_clk_en**: bool 属性，表示使用非连续 clock，即在 Hporch/Vporch 阶段，或 DPU 停止送数阶段，clock lane 可以自动进入 LP11 状态，以节省功耗。




