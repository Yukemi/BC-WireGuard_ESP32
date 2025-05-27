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
- iperf socket TCP/UDP rx timeout in seconds
- iperf socket TCP tx timeout in seconds

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
```

## Log output example

```console

```
