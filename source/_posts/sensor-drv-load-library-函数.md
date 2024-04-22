---
title: sensor_drv_load_library()函数
tags:
  - 展锐
  - camera
  - sns_drv_u
abbrlink: 28b0fb00
date: 2024-04-02 17:46:14
---

函数返回的结构体内容如下：

```c
struct sensor_drv_lib {
    void *drv_lib_handle;
    SENSOR_INFO_T *sensor_info_ptr;
};
```

这段代码是一个用于加载动态链接库的函数，其功能是根据给定的传感器名称（`name`）加载对应的动态链接库，并获取其中的函数指针。

```c
static cmr_int sensor_drv_load_library(const char *name,
                                       struct sensor_drv_lib *libPtr) {
    cmr_int ret = SENSOR_FAIL;
    char libso_name[SENSOR_LIB_NAME_LEN] = {0};

    void *(*sensor_ic_open_lib)(void) = NULL;
    int32_t bytes = 0;

    if (!strlen(name)) {
        SENSOR_LOGE("don't config sensor in xml file");
        goto exit;
    }

    bytes = snprintf(libso_name, SENSOR_LIB_NAME_LEN, "libsensor_%s.so", name);
    SENSOR_LOGD("sensor:libso_name %s", libso_name);
    libPtr->drv_lib_handle = dlopen(libso_name, RTLD_NOW);
    if (!libPtr->drv_lib_handle) {
        SENSOR_LOGE("sensor lib handle failed");
        goto exit;
    }

    *(void **)&sensor_ic_open_lib =
        dlsym(libPtr->drv_lib_handle, "sensor_ic_open_lib");
    if (!sensor_ic_open_lib) {
        SENSOR_LOGE("sensor ic open lib function failed");
        dlclose(libPtr->drv_lib_handle);
        libPtr->drv_lib_handle = NULL;
        goto exit;
    }
    libPtr->sensor_info_ptr = (SENSOR_INFO_T *)sensor_ic_open_lib();
    if (!libPtr->sensor_info_ptr) {
        SENSOR_LOGE("load sensor_info_ptr failed");
        dlclose(libPtr->drv_lib_handle);
        libPtr->drv_lib_handle = NULL;
        goto exit;
    }

    ret = 0;
exit:

    return ret;
}
```

以打开 `libsensor_imx351.so` 库为例：
```c
void *sensor_ic_open_lib(void)
{
    return &g_imx351_mipi_raw_info;
}
```

`g_imx351_mipi_raw_info` 内容如下：
```c
SENSOR_INFO_T g_imx351_mipi_raw_info = {
    .hw_signal_polarity = SENSOR_HW_SIGNAL_PCLK_P | SENSOR_HW_SIGNAL_VSYNC_P |
                          SENSOR_HW_SIGNAL_HSYNC_P,
    .environment_mode = SENSOR_ENVIROMENT_NORMAL | SENSOR_ENVIROMENT_NIGHT,
    .image_effect = SENSOR_IMAGE_EFFECT_NORMAL |
                    SENSOR_IMAGE_EFFECT_BLACKWHITE | SENSOR_IMAGE_EFFECT_RED |
                    SENSOR_IMAGE_EFFECT_GREEN | SENSOR_IMAGE_EFFECT_BLUE |
                    SENSOR_IMAGE_EFFECT_YELLOW | SENSOR_IMAGE_EFFECT_NEGATIVE |
                    SENSOR_IMAGE_EFFECT_CANVAS,

    .wb_mode = 0,
    .step_count = 7,
    .reset_pulse_level = SENSOR_LOW_PULSE_RESET,
    .reset_pulse_width = 50,
    .power_down_level = SENSOR_LOW_LEVEL_PWDN,
    .identify_count = 1,
    .identify_code = {{.reg_addr = imx351_PID_ADDR,
                       .reg_value = imx351_PID_VALUE},
                      {.reg_addr = imx351_VER_ADDR,
                       .reg_value = imx351_VER_VALUE}},

    .source_width_max = SNAPSHOT_WIDTH,
    .source_height_max = SNAPSHOT_HEIGHT,
    .name = (cmr_s8 *)SENSOR_NAME,
    .image_format = SENSOR_IMAGE_FORMAT_RAW,

    .resolution_tab_info_ptr = s_imx351_resolution_tab_raw,
    .sns_ops = &s_imx351_ops_tab,
    .raw_info_ptr = &s_imx351_mipi_raw_info_ptr,
    .module_info_tab = s_imx351_module_info_tab,
    .module_info_tab_size = ARRAY_SIZE(s_imx351_module_info_tab),
    .video_tab_info_ptr = s_imx351_video_info,
    .ext_info_ptr = NULL,

    .sensor_version_info = (cmr_s8 *)"imx351v1"};
```

`sns_drv_u` 模块有大量结构体包含`SENSOR_INFO_T`成员，这个成员就是`ic`驱动的 `g_[sensor ic]_mipi_raw_info` 结构体。