idf_component_register(SRCS 
                        "main.c" 
                        "sync_time.c"
                        "../components/ESP32_Task_Monitor/src/task_monitor.c"
                        INCLUDE_DIRS 
                        "."
                        "../components/ESP32_Task_Monitor/include"
                    REQUIRES 
                        esp_wifi 
                        lwip 
                        esp_wireguard 
                        nvs_flash 
                        esp_driver_tsens # Temp sensor
                    )
