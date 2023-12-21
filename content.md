---
title: 'Connect the Finder Opta to the Internet via Ethernet'
description: "Learn how to configure an IP address on the Finder Opta to
              connect it to the Internet via Ethernet"
author: 'Fabrizio Trovato'
difficulty: easy
tags:
  - Ethernet
software:
  - ide-v1
  - ide-v2
  - arduino-cli
  - web-editor
hardware:
  - hardware/07.opta/opta-family/opta
---

## Overview

In this tutorial, we will learn how to connect the Finder Opta to the Internet
via an Ethernet channel, configuring a static or a dynamic IP address. In
particular, we will first show how to manually configure the IP address and the
DNS, and then we will show how to configure them automatically using DHCP.

## Goals

* Learn how to assign a static IP addrress to the Finder Opta.
* Learn how to assign a dynamic IP addrress to the Finder Opta.

## Required Hardware and Software

### Hardware Requirements

* Finder Opta PLC (x1).
* USB-C® cable (x1).
* ETH RJ45 cable (x1).

### Software Requirements

* [Arduino IDE 1.8.10+](https://www.arduino.cc/en/software), [Arduino IDE
2.0+](https://www.arduino.cc/en/software) or [Arduino Web
Editor](https://create.arduino.cc/editor).
* [Example code with static IP address](assets/OptaEthernetStatic.zip).
* [Example code with DHCP](assets/OptaEthernetDHCP.zip).

## Finder Opta and Ethernet

Using the `EthernetClient` class, the Finder Opta can configure its network
parameters and verify its connectivity to the Internet.

## Instructions

### Setting Up the Arduino IDE

This tutorial will need [the latest version of the Arduino
IDE](https://www.arduino.cc/en/software). If it is your first time setting up
the Finder Opta, check out the [getting started
tutorial](/tutorials/opta/getting-started).

### Connectivity

The only requirement for this tutorial is that the Finder Opta must be
connected via Ethernet to a device that can assign it a dynamic IP address
using DHCP, and that can later route the packets from the Finder Opta to the
HTTP server and viceveversa.

### Code Overview

The goal of the following tutorial is to configure the network parameters of
the Finder Opta, first statically and then dinamically. This means that the
Finder Opta will then be connected to the Internet, allowing us to perform some
connectivity checks.

The full code of the example using a static IP address is available
[here](assets/OptaHttpClientTutorial.zip), while you can find the version using
DHCP at [this link](assets/OptaEthernetDHCP.zip). After extracting the files
the sketch can be compiled and uploaded to the Finder Opta.

#### Configuration with a static IP address

In this case it will be necessary to configure the Finder Opta with a static IP
address belonging to the same LAN of the router, and then to specify the IP
address if the DNS server we want to use. Once the variables are defined, it
will be enough to provide the MAC address of the device to the `begin()`
function of the `Ethernet` class.

```cpp
OptaBoardInfo *info;
OptaBoardInfo *boardInfo();
EthernetClient client;

const char server[] = "example.com";
IPAddress ip(192, 168, 10, 15);
IPAddress dns(192, 168, 10, 1);

void setup()
{
    info = boardInfo();
    // Check if secure informations are available since MAC Address is among them.
    if (info->magic = 0xB5)
    {
        // Assign static IP address.
        Ethernet.begin(info->mac_address, ip, dns);
    }
    else
    {
        digitalWrite(LED_D1, HIGH);
        while (1)
        {
        }
    }
}
```

If the sketch cannot read the MAC address the LED number 1 will turn on,
otherwise our Finder Opta will have an IP address and a DNS server configured,
allowing us to proceed with the execution of the program.

#### Configuration with a dynamic IP address

In this case it will be enough to provide the MAC address of the Finder Opta to
the `begin()` function, which will take care of getting the dynamic IP address
and the IP address of the DNS server via DHCP:

```cpp
OptaBoardInfo *info;
OptaBoardInfo *boardInfo();
EthernetClient client;

const char server[] = "example.com";

void setup()
{
    info = boardInfo();
    // Check if secure informations are available since MAC Address is among them.
    if (info->magic = 0xB5)
    {
        // Attempt DHCP lease.
        if (Ethernet.begin(info->mac_address) == 0)
        {
            digitalWrite(LED_D0, HIGH);
            while (1)
            {
            }
        }
    }
    else
    {
        digitalWrite(LED_D1, HIGH);
        while (1)
        {
        }
    }
}
```

If the sketch cannot read the MAC address the LED number 1 will turn on, while
if instead there are problems during the DHCP lease the LED number 0 will turn
on. In case we have no errors, our Finder Opta will have an IP address and a
DNS server configured, allowing us to proceed with the execution of the
program.

#### Connectivity check

Below we find the code of the `loop()` function:

```cpp
void loop()
{
    // Connect to the server and turn on user LED to show result.
    if (client.connect(server, 80))
    {
        digitalWrite(LED_D3, HIGH);
    }
    else
    {
        digitalWrite(LED_D2, HIGH);
    }
    // Do nothing for a minute.
    delay(60000);
    digitalWrite(LED_D3, LOW);
    digitalWrite(LED_D2, LOW);
}

```

This function performs a connectivity checks towards port 80 of the server
specified in the sketch, turning on LED 3 in case of success and LED 2 in case
of failure.

## Conclusion

This tutorial shows how to assign a static or dynamic IP address to the Finder
Opta, then it performs a connectivity check to verify that the device can
communicate via Ethernet with a remote server.
