# demo example

## What the example does

The example connects to a WireGuard server. When the link is up, the device
sends ICMP echo requests, and shows ping statistics. The session resets after 60 seconds
forever.

The main task then disconnects from the peer, and re-connects to the peer.

## Requirements

* An ESP32 or ESP8266 development board
* WiFi network
* [`wireguard-tools`](https://github.com/WireGuard/wireguard-tools)
* A WireGuard server

## Generating keys

```console
wg genkey | tee private.key | wg pubkey > public.key
```

## Log

```console

```
