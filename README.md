# Wireguard performance monitoring for ESP32

## Project description

The project sets up a WireGuard connection, and a Iperf server to test network speed within and outside the WireGuard tunnel between the ESP32 and a client machine.

The project includes a Task Monitor to report runtime, and stack size of each running task, along with total free heap size and temperature.

The Iperf socket closes after a single test has been perfomed, because of that
it runs on limited time then restarts (adjustable in `menuconfig`)

The project is based on the example demo of [`WireGuard for ESP32-S3`](https://github.com/trombik/esp_wireguard/tree/main/examples/demo)

## Requirements

* An ESP32
* WiFi network
* [`WireGuard for Windows`](https://www.wireguard.com/install/)
* A WireGuard server
* Compatible version of [`iperf`](https://sourceforge.net/projects/iperf2/) (v2.x.x is recommended)

## Getting Started

Clone the repository
```bash
git clone https://github.com/Yukemi/BC-WireGuard_ESP32.git
```





## Log

```console

```
