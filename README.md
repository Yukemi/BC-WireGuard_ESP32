# demo example

## What the example does

The example connects to a WireGuard server. When the link is up, the device
listens for a iperf test, and reports network bandwidth. The session resets after specified ammount of time.
(60 seconds by default, can be adjusted in menuconfig)

## Requirements

* An ESP32
* WiFi network
* [`wireguard-tools`](https://github.com/WireGuard/wireguard-tools)
* A WireGuard server
* Compatible version of [`iperf`](https://sourceforge.net/projects/iperf2/) (v2.x.x is recommended)

## Generating keys

```console
wg genkey | tee private.key | wg pubkey > public.key
```

## Log

```console

```
