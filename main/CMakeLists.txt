idf_component_register(SRCS 
                        "main.c" 
                        "sync_time.c"
                        "../components/ESP32_Task_Monitor/src/task_monitor.c"   #task monitor
                        "../components/iperf/iperf/iperf.c"                     #iperf
                        "../components/iperf/iperf-cmd/iperf_cmd.c"             #iperf
                        INCLUDE_DIRS 
                        "."
                        "../components/ESP32_Task_Monitor/include"              #task monitor
                        "../components/iperf/iperf/include"                     #iperf
                        "../components/iperf/iperf-cmd/include"                 #iperf
                    REQUIRES 
                        esp_wifi 
                        lwip 
                        esp_wireguard 
                        nvs_flash 
                        esp_driver_tsens # Temp sensor
                    )

# Iperf
set(EXTRA_COMPONENT_DIRS components/iperf)
include($ENV{IDF_PATH}/tools/cmake/project.cmake)
