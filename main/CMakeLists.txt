idf_component_register(SRCS 
                        "main.c" 
                        "sync_time.c"
                        "../src/task_monitor.c"
                    INCLUDE_DIRS 
                        "."
                        "../include"
                    REQUIRES 
                        esp_wifi 
                        lwip 
                        esp_wireguard 
                        nvs_flash 
                        esp_driver_tsens # Temp sensor
                    )
