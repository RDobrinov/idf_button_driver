# IDF Button Driver

Button control driver for ESP IDF. Tested with the ESP32, ESP32-S3, and ESP32-C6 boards.

![](https://img.shields.io/badge/dynamic/yaml?url=https://raw.githubusercontent.com/RDobrinov/idf_button_driver/main/idf_component.yml&query=$.version&style=plastic&color=%230f900f&label)
![](https://img.shields.io/badge/dynamic/yaml?url=https://raw.githubusercontent.com/RDobrinov/idf_button_driver/main/idf_component.yml&query=$.dependencies.idf&style=plastic&logo=espressif&label=IDF%20Ver.)
![](https://img.shields.io/badge/-ESP32-rgb(37,194,160)?style=plastic&logo=espressif)
![](https://img.shields.io/badge/-ESP32--S3-rgb(37,194,160)?style=plastic&logo=espressif)
![](https://img.shields.io/badge/-ESP32--C6-rgb(37,194,160)?style=plastic&logo=espressif)

---

## Features

* Unlimited number of controlled buttons. Maximum number is number of unclaimed board pins
* Can handle up to three clicks and/or long/very long click
* Periodically denerate __HOLD__ event in case of button holding. 
* Click intervals can be controled via configuration parameters
* Event notification via __default__ or __user created__ event loop 
* The driver runs as a dedicated task

## Installation

1. Create *idf_component.yml*
```
idf.py create-manifest
```
2. Edit ***idf_component.yml*** to add dependency
```
dependencies:
  ...
  idf_wifi_manager:
    version: "main"
    git: git@github.com:RDobrinov/idf_button_driver.git
  ...
```
3. Reconfigure project

or 

4. Download and unzip component in project ***components*** folder

### Example
```
#include <stdio.h>
#include "idf_button_driver.h"

vvoid btndrv_event_handler(void* arg, esp_event_base_t event_base, int32_t event_id, void* event_data) {
    /* Do nothing */
    return;
}

void app_main(void)
{
    esp_event_loop_handle_t *uevent_loop = (esp_event_loop_handle_t *)malloc(sizeof(esp_event_loop_handle_t));
    esp_event_loop_args_t uevent_args = {
        .queue_size = 5,
        .task_name = "uevloop",
        .task_priority = 15,
        .task_stack_size = 3072,
        .task_core_id = tskNO_AFFINITY
    };
    esp_event_loop_create(&uevent_args, uevent_loop);
    esp_event_handler_instance_register_with(*uevent_loop, BTNDRV_EVENT, ESP_EVENT_ANY_ID, btndrv_event_handler, NULL, NULL);
    
    btn_drv_init(uevent_loop);

    btn_drv_config_t *boot_btn = (btn_drv_config_t *)malloc(sizeof(btn_drv_config_t));
    boot_btn->btn_gpio_num = GPIO_NUM_9;
    boot_btn->button_id = 0x00;
    boot_btn->def_state = 0x01;
    boot_btn->btn_gpio_mode = GPIO_PULLUP_ONLY;

    esp_event_post_to(*uevent_loop, BTNDRV_EVENT, BTNDRV_EVENT_REG_BUTTON, boot_btn, sizeof(btn_drv_config_t), 1);
    boot_btn->btn_gpio_num = GPIO_NUM_4;
    esp_event_post_to(*uevent_loop, BTNDRV_EVENT, BTNDRV_EVENT_REG_BUTTON, boot_btn, sizeof(btn_drv_config_t), 1);
    boot_btn->btn_gpio_num = GPIO_NUM_5;
    esp_event_post_to(*uevent_loop, BTNDRV_EVENT, BTNDRV_EVENT_REG_BUTTON, boot_btn, sizeof(btn_drv_config_t), 1);

    free(boot_btn);
}
```
