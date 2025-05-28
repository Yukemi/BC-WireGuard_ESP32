# Wireguard performance monitoring for ESP32

## Project description

The project sets up a WireGuard connection, and a Iperf server to test network speed within and outside the WireGuard tunnel between the ESP32 and a client machine.

The project includes a Task Monitor to report runtime, and stack size of each running task, along with total free heap size and temperature.

The Iperf socket closes after a single test has been perfomed, because of that
it runs on limited time then restarts (adjustable in `menuconfig`)

The project is based on the example demo of [`WireGuard for ESP32-S3`](https://github.com/trombik/esp_wireguard/tree/main/examples/demo)

The task monitor code is taken from [`ESP32_Task_Monitor`](https://github.com/VPavlusha/ESP32_Task_Monitor) and included code to report the chip temperature.



## Requirements

* An ESP32
* WiFi network
* [`WireGuard for Windows`](https://www.wireguard.com/install/)
* A WireGuard server
* Compatible version of [`iperf`](https://sourceforge.net/projects/iperf2/) (v2.x.x is recommended)



## Getting Started

### Key generation
Make sure to make keys for both the server and the ESP32 (except for `preshared.key`)
```bash
wg genkey | tee private.key | wg pubkey > public.key
wg genpsk > preshared.key
```

### Clone the repository
```bash
git clone https://github.com/Yukemi/BC-WireGuard_ESP32.git
```

### Set target
```bash
cd BC-WireGuard_ESP32
idf.py set-target <target>
```

### Configure the project
```bash
idf.py menuconfig
```
Head to `Component config` -> `FreeRTOS` -> `Kernel`

Enable for `ESP32_Task_Monitor` to work:

* `configUSE_TRACE_FACILITY`
* `configUSE_STATS_FORMATTING_FUNCTIONS`
* `Enable display of xCoreID in vTaskList`
* `configGENERATE_RUN_TIME_STATS`

Enable PPP Support in `Component Config` -> `LWIP` -> `Enable PPP Support`

Then set up the rest in `WireGuard Configuration` and `Iperf configuration`

Adjust socket timeouts in `Component Config` -> `iperf`, increase the timers to avoid the socket closing up
* `iperf socket TCP/UDP rx timeout in seconds`
* `iperf socket TCP tx timeout in seconds`

See [Project Configuration Example](#project-configuration-example) for more information

### Build, flash, and monitor the project
```bash
idf.py -p <port> [-b <baud rate>] [clean] flash monitor
```

If you're using VSCode, you can use the quick buttons ESP-IDF comes with to do the above.

### Run Iperf test
For socket buffer size you can use for example: `128K`
```bash
.\iperf.exe -c <ESP32 IP> -i <Report Interval> -t <Test time> -w <Socket Buffer Size> -p <destination port>
```


## Project Configuration Example
> WireGuard (Windows Client)
```conf
[Interface]
PrivateKey   = private.key                         (Server key)
ListenPort   = listening port                      (eg. 11010)
Address      = IP within tunnel                    (eg. 10.0.0.1/24)

[Peer]
PublicKey    = public.key                          (Client key)
PresharedKey = preshared.key
AllowedIPs   = Allowed IP range                    (eg. 10.0.0.0/24)
Endpoint     = [ESP32 IP address]:[listening port]
```
> ESP32 (sdkconfig)
```conf
#
# WireGuard Configuration
#
CONFIG_ESP_WIFI_SSID="[Network Name]"
CONFIG_ESP_WIFI_PASSWORD="[Network Pass]"
CONFIG_ESP_MAXIMUM_RETRY=5
CONFIG_WG_PRIVATE_KEY="[private.key (ESP32 key)]"
CONFIG_WG_LOCAL_IP_ADDRESS="[ESP IP address within tunnel]"
CONFIG_WG_LOCAL_IP_NETMASK="255.255.255.0"
CONFIG_WG_LOCAL_PORT=11010
CONFIG_WG_PEER_PUBLIC_KEY="[public.key (server key)]"
CONFIG_WG_PRESHARED_KEY="[preshared.key]"
CONFIG_WG_PEER_ADDRESS="[Server IP address]"
CONFIG_WG_PEER_PORT=11010
CONFIG_WG_PERSISTENT_KEEP_ALIVE=0
# end of WireGuard Configuration

#
# Iperf configuration
#
CONFIG_IPERF_SOURCE_IP_ADDRESS=0
CONFIG_IPERF_SOURCE_IP_PORT=5201
CONFIG_IPERF_DESTINATION_IP_PORT=5201
CONFIG_IPERF_LEN_SEND_BUFFER=16384
CONFIG_IPERF_INTERVAL=3
CONFIG_IPERF_TIME=30
CONFIG_IPERF_LOOP_RESET_ADD=30
# end of Iperf configuration

#
# Kernel
#
...
CONFIG_FREERTOS_USE_TRACE_FACILITY=y
CONFIG_FREERTOS_USE_STATS_FORMATTING_FUNCTIONS=y
# CONFIG_FREERTOS_USE_LIST_DATA_INTEGRITY_CHECK_BYTES is not set
CONFIG_FREERTOS_VTASKLIST_INCLUDE_COREID=y
CONFIG_FREERTOS_GENERATE_RUN_TIME_STATS=y
CONFIG_FREERTOS_RUN_TIME_COUNTER_TYPE_U32=y
...
# end of Kernel

#
# Iperf
#
CONFIG_IPERF_SOCKET_RX_TIMEOUT=30
CONFIG_IPERF_SOCKET_TCP_TX_TIMEOUT=30
...
# end of Iperf
```

## Log output example

```console
I (27) boot: ESP-IDF v5.4.1 2nd stage bootloader
I (27) boot: compile time May 27 2025 21:09:36
I (27) boot: Multicore bootloader
I (27) boot: chip revision: v0.1
I (30) boot: efuse block revision: v1.2
I (33) boot.esp32s3: Boot SPI Speed : 80MHz
I (37) boot.esp32s3: SPI Mode       : DIO
I (41) boot.esp32s3: SPI Flash Size : 2MB
I (45) boot: Enabling RNG early entropy source...
I (49) boot: Partition Table:
I (52) boot: ## Label            Usage          Type ST Offset   Length
I (58) boot:  0 nvs              WiFi data        01 02 00009000 00006000
I (65) boot:  1 phy_init         RF data          01 01 0000f000 00001000
I (71) boot:  2 factory          factory app      00 00 00010000 00100000
I (78) boot: End of partition table
I (81) esp_image: segment 0: paddr=00010020 vaddr=3c0a0020 size=1d518h (120088) map
I (110) esp_image: segment 1: paddr=0002d540 vaddr=3fc9a600 size=02ad8h ( 10968) load
I (112) esp_image: segment 2: paddr=00030020 vaddr=42000020 size=97aa4h (621220) map
I (223) esp_image: segment 3: paddr=000c7acc vaddr=3fc9d0d8 size=02028h (  8232) load
I (225) esp_image: segment 4: paddr=000c9afc vaddr=40374000 size=165f0h ( 91632) load
I (247) esp_image: segment 5: paddr=000e00f4 vaddr=600fe100 size=0001ch (    28) load
I (256) boot: Loaded app from partition at offset 0x10000
I (256) boot: Disabling RNG early entropy source...
I (266) cpu_start: Multicore app
I (275) cpu_start: Pro cpu start user code
I (275) cpu_start: cpu freq: 160000000 Hz
I (275) app_init: Application information:
I (275) app_init: Project name:     demo
I (279) app_init: App version:      ac41993
I (283) app_init: Compile time:     May 27 2025 22:34:10
I (288) app_init: ELF file SHA256:  e3ceb4670...
I (292) app_init: ESP-IDF:          v5.4.1
I (296) efuse_init: Min chip rev:     v0.0
I (300) efuse_init: Max chip rev:     v0.99 
I (304) efuse_init: Chip rev:         v0.1
I (308) heap_init: Initializing. RAM available for dynamic allocation:
I (314) heap_init: At 3FCA3830 len 00045EE0 (279 KiB): RAM
I (319) heap_init: At 3FCE9710 len 00005724 (21 KiB): RAM
I (324) heap_init: At 3FCF0000 len 00008000 (32 KiB): DRAM
I (329) heap_init: At 600FE11C len 00001ECC (7 KiB): RTCRAM
I (336) spi_flash: detected chip: generic
I (338) spi_flash: flash io: dio
W (341) spi_flash: Detected size(8192k) larger than the size in the binary image header(2048k). Using the size in the binary image header.
I (354) sleep_gpio: Configure to isolate all GPIO pins in sleep state
I (360) sleep_gpio: Enable automatic switching of GPIO sleep configuration
I (367) main_task: Started on CPU0
I (387) main_task: Calling app_main()
I (407) demo: Event group handle created
I (407) demo: Event group handle: 0x3fca974c
I (407) pp: pp rom version: e7ae62f
I (407) net80211: net80211 rom version: e7ae62f
I (417) wifi:wifi driver task: 3fcae428, prio:23, stack:6656, core=0
I (427) wifi:wifi firmware version: 79fa3f41ba
I (427) wifi:wifi certification version: v7.0
I (427) wifi:config NVS flash: enabled
I (427) wifi:config nano formatting: disabled
I (427) wifi:Init data frame dynamic rx buffer num: 32
I (437) wifi:Init static rx mgmt buffer num: 5
I (437) wifi:Init management short buffer num: 32
I (447) wifi:Init dynamic tx buffer num: 32
I (447) wifi:Init static tx FG buffer num: 2
I (447) wifi:Init static rx buffer size: 1600
I (457) wifi:Init static rx buffer num: 10
I (457) wifi:Init dynamic rx buffer num: 32
I (467) wifi_init: rx ba win: 6
I (467) wifi_init: accept mbox: 6
I (467) wifi_init: tcpip mbox: 32
I (477) wifi_init: udp mbox: 6
I (477) wifi_init: tcp mbox: 6
I (477) wifi_init: tcp tx win: 5760
I (477) wifi_init: tcp rx win: 5760
I (487) wifi_init: tcp mss: 1440
I (487) wifi_init: WiFi IRAM OP enabled
I (487) wifi_init: WiFi RX IRAM OP enabled
I (497) phy_init: phy_version 700,8582a7fd,Feb 10 2025,20:13:11
I (537) wifi:mode : sta (...)
I (537) wifi:enable tsf
I (647) wifi:new:<7,0>, old:<1,0>, ap:<255,255>, sta:<7,0>, prof:1, snd_ch_cfg:0x0
I (647) wifi:state: init -> auth (0xb0)
I (647) wifi:state: auth -> assoc (0x0)
I (657) wifi:state: assoc -> run (0x10)
I (657) wifi:state: run -> init (0x2c0)
I (667) wifi:new:<7,0>, old:<7,0>, ap:<255,255>, sta:<7,0>, prof:1, snd_ch_cfg:0x0
I (667) demo: retry to connect to the AP
I (667) demo: connect to the AP fail
I (3077) demo: retry to connect to the AP
I (3077) demo: connect to the AP fail
I (3077) wifi:new:<7,0>, old:<7,0>, ap:<255,255>, sta:<7,0>, prof:1, snd_ch_cfg:0x0
I (3087) wifi:state: init -> auth (0xb0)
I (3097) wifi:state: auth -> assoc (0x0)
I (3107) wifi:state: assoc -> run (0x10)
I (3127) wifi:connected with ..., aid = 17, channel 7, BW20, bssid = ...
I (3127) wifi:security: WPA2-PSK, phy: bgn, rssi: -51
I (3127) wifi:pm start, type: 1

I (3127) wifi:dp: 1, bi: 102400, li: 3, scale listen interval from 307200 us to 307200 us
I (3137) wifi:set rx beacon pti, rx_bcn_pti: 0, bcn_timeout: 25000, mt_pti: 0, mt_time: 10000
I (3157) wifi:<ba-add>idx:0 (ifx:0, ...), tid:5, ssn:2, winSize:64
I (3197) wifi:AP's beacon interval = 102400 us, DTIM period = 1
I (4167) esp_netif_handlers: sta ip: ..., mask: 255.255.255.0, gw: ...
I (4167) demo: got ip:...
I (4167) demo: Connected to ap SSID:...
I (4167) demo: Starting iperf server
I (4167) sync_time: Initializing SNTP
I (4167) iperf: Socket created
I (4177) sync_time: Waiting for system time to be set... (1/20)
I (6187) sync_time: Waiting for system time to be set... (2/20)
I (8187) sync_time: Waiting for system time to be set... (3/20)
I (8837) wifi:<ba-add>idx:1 (ifx:0, ...), tid:0, ssn:0, winSize:64
I (8857) sync_time: Time synced
I (10187) demo: The current date/time in New York is: Wed May 28 10:16:31 2025
I (10187) demo: Initializing WireGuard.
I (10187) demo: Connecting to the peer.
I (10187) esp_wireguard: allowed_ip: 10.0.0.2
I (10237) esp_wireguard: using preshared_key
I (10237) esp_wireguard: X25519: default
I (10237) esp_wireguard: Peer: ... (...:11010)
I (10277) esp_wireguard: Connecting to ...:11010
I (11277) demo: Peer is up
I (10910) tm: task_status_monitor_task() started successfully
I (11277) temp_monitor: Install temperature sensor, expected temp ranger range: 10~50 ℃
I (11277) temperature_sensor: Range [-10°C ~ 80°C], error < 1°C
I (11287) temp_monitor: Enable temperature sensor
I (11287) temp_monitor: Read temperature

I (122200) tm: TASK NAME:         STATE:      CORE:  NUMBER:  PRIORITY:  STACK_MIN:  RUNTIME, µs:   RUNTIME, %:
I (122200) tm: IDLE0              Ready       0      5        0          688         115308439      94.339    
I (122210) tm: wifi               Blocked     0      10       23         3464        4412485        3.610     
I (122220) tm: monitor_task       Running     0      13       1          3924        1963656        1.607     
I (122230) tm: main               Blocked     0      4        1          1192        186279         0.152     
I (122240) tm: esp_timer          Suspended   0      3        22         3088        83464          0.068
I (122250) tm: ipc0               Suspended   0      1        24         528         32199          0.026     
I (122260) tm: sys_evt            Blocked     0      9        20         580         2025           0.002     
I (122270) tm: IDLE1              Ready       1      6        0          696         117842183      96.412    
I (122280) tm: tiT                Blocked     Any    8        18         1212        3390081        2.774     
I (122290) tm: ipc1               Suspended   1      2        24         536         56903          0.047     
I (122300) tm: iperf_server       Blocked     Any    11       3          2016        4631           0.004     
I (122310) tm: Tmr Svc            Blocked     Any    7        1          1312        18             0.000
I (122687) temp_monitor: Temperature value 35.60 °C
I (122330) tm: Total heap size:        349392 bytes
I (122340) tm: Current heap free size: 259904 bytes (74.39 %)
I (122340) tm: Minimum heap free size: 222988 bytes (63.82 %)
I (122350) tm: Total RunTime: 122228064 µs (122 seconds)
I (122360) tm: System UpTime: 122394107 µs (122 seconds)

I (126427) demo: Stopping Iperf
I (126427) iperf: DONE.IPERF_STOP,OK.
I (127427) demo: Starting Iperf
I (127427) demo: Starting iperf server
I (127427) iperf: Socket created
I (128337) iperf: accept: 192.168.0.107,54640

I (152750) tm: TASK NAME:         STATE:      CORE:  NUMBER:  PRIORITY:  STACK_MIN:  RUNTIME, µs:   RUNTIME, %:
I (152750) tm: IDLE0              Ready       0      5        0          688         140926219      92.242    
I (152760) tm: wifi               Blocked     0      10       23         3464        8787975        5.752     
I (152770) tm: monitor_task       Running     0      13       1          3924        2454214        1.606
I (152780) tm: main               Blocked     0      4        1          1192        186354         0.122     
I (152790) tm: esp_timer          Suspended   0      3        22         3088        96368          0.063     
I (152800) tm: ipc0               Suspended   0      1        24         528         32199          0.021     
I (152810) tm: sys_evt            Blocked     0      9        20         580         2025           0.001
I (152820) tm: IDLE1              Ready       1      6        0          696         142888922      93.527    
I (152830) tm: tiT                Blocked     Any    8        18         1212        7447256        4.875     
I (152840) tm: iperf_traffic      Blocked     1      21       4          1896        1490913        0.976
I (152850) tm: ipc1               Suspended   1      2        24         536         56903          0.037     
I (152860) tm: iperf_report       Blocked     1      22       6          2156        6972           0.005     
I (152880) tm: iperf_server       Blocked     Any    11       3          2016        6025           0.004
I (152890) tm: Tmr Svc            Blocked     Any    7        1          1312        18             0.000     
I (153267) temp_monitor: Temperature value 39.60 °C
I (152900) tm: Total heap size:        349392 bytes
I (152910) tm: Current heap free size: 233416 bytes (66.81 %)
I (152920) tm: Minimum heap free size: 222988 bytes (63.82 %)
I (152920) tm: Total RunTime: 152778247 µs (152 seconds)
I (152930) tm: System UpTime: 152966234 µs (152 seconds)

Interval       Bandwidth
 0.0- 3.0 sec  1.55 Mbits/sec
 3.0- 6.0 sec  2.68 Mbits/sec
 6.0- 9.0 sec  2.79 Mbits/sec
 9.0-12.0 sec  2.52 Mbits/sec
12.0-15.0 sec  2.60 Mbits/sec
15.0-18.0 sec  2.81 Mbits/sec
18.0-21.0 sec  2.57 Mbits/sec
21.0-24.0 sec  3.04 Mbits/sec
24.0-27.0 sec  2.65 Mbits/sec
27.0-30.0 sec  2.63 Mbits/sec
 0.0-30.0 sec  2.58 Mbits/sec
I (158457) iperf: TCP Socket server is closed.
I (158457) iperf: iperf exit

```

## Iperf output example

```console
> .\iperf.exe -c ... -i 3 -t 30 -w 128k -p 5201
------------------------------------------------------------
Client connecting to ..., TCP port 5201
TCP window size:  125 KByte
------------------------------------------------------------
[  1] local ... port 54640 connected with ... port 5201
[ ID] Interval       Transfer     Bandwidth
[  1] 0.00-3.00 sec   768 KBytes  2.10 Mbits/sec
[  1] 3.00-6.00 sec  1.00 MBytes  2.80 Mbits/sec
[  1] 6.00-9.00 sec  1.00 MBytes  2.80 Mbits/sec
[  1] 9.00-12.00 sec  1.00 MBytes  2.80 Mbits/sec
[  1] 12.00-15.00 sec  1.00 MBytes  2.80 Mbits/sec
[  1] 15.00-18.00 sec  1.00 MBytes  2.80 Mbits/sec
[  1] 18.00-21.00 sec  1.00 MBytes  2.80 Mbits/sec
[  1] 21.00-24.00 sec  1.13 MBytes  3.15 Mbits/sec
[  1] 24.00-27.00 sec  1.00 MBytes  2.80 Mbits/sec
[  1] tcp write failed
[  1] 27.00-30.00 sec  1.00 MBytes  2.80 Mbits/sec
[  1] 0.00-30.13 sec  9.88 MBytes  2.75 Mbits/sec
```
