idf_component_register(SRCS 
                        "main.c" 
                        "sync_time.c"
                    INCLUDE_DIRS 
                        "."
                    REQUIRES 
                        esp_wifi                                                #wireguard
                        lwip                                                    #wireguard
                        esp_wireguard                                           #wireguard
                        nvs_flash                                               #wireguard
                        esp_driver_tsens                                        #Temp sensor
                        console                                                 #iperf
                    )