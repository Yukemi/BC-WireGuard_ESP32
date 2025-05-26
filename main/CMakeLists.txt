idf_component_register(SRCS 
                        "main.c" 
                        "sync_time.c"
                        "../components/ESP32_Task_Monitor/src/task_monitor.c"   #task monitor
                        INCLUDE_DIRS 
                        "."
                        "../components/ESP32_Task_Monitor/include"              #task monitor
                    REQUIRES 
                        esp_wifi                                                #wireguard
                        lwip                                                    #wireguard
                        esp_wireguard                                           #wireguard
                        nvs_flash                                               #wireguard
                        esp_driver_tsens                                        #Temp sensor
                        console                                                 #iperf
                        esp_timer                                               #task monitor
                    )