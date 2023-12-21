---
title: 'Connettere Finder Opta ad Internet tramite Ethernet'
description: "Imparare a configurare un indirizzo IP su Finder Opta per
              connetterlo ad Internet tramite Ethernet."
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

## Panoramica

In questo tutorial, impareremo a connettere Finder Opta ad Internet tramite un
canale Ethernet, andando a configurare un indirizzo IP statico o dinamico. In
particolare, mostreremo prima come configurare indirizzo IP e DNS manualmente,
ed in seguito come configurarli tramite DHCP.

## Obiettivi

* Imparare ad assegnare un indirizzo IP statico a Finder Opta.
* Imparare ad assegnare un indirizzo IP dinamico a Finder Opta.

## Requisiti hardware e software

### Requisiti hardware

* PLC Finder Opta (x1).
* Cavo USB-C® (x1).
* Cavo ETH RJ45 (x1).

### Requisiti Software

* [Arduino IDE 1.8.10+](https://www.arduino.cc/en/software), [Arduino IDE
2.0+](https://www.arduino.cc/en/software) o [Arduino Web
Editor](https://create.arduino.cc/editor).
* [Codice di esempio con indirizzo IP statico](assets/OptaEthernetStatic.zip).
* [Codice di esempio con DHCP](assets/OptaEthernetDHCP.zip).

## Finder Opta ed Ethernet

Grazie alla classe `EthernetClient`, Finder Opta può configurare le proprie
impostazioni di rete e verificare il proprio stato di connettività.

## Istruzioni

### Configurazione dell'Arduino IDE

Per seguire questo tutorial, sarà necessaria [l'ultima versione dell'Arduino
IDE](https://www.arduino.cc/en/software). Se è la prima volta che configuri il
Finder Opta, dai un'occhiata al tutorial [Getting Started with
Opta](/tutorials/opta/getting-started).

### Connettività

L'unico requisito di questo tutorial è che Finder Opta sia connesso tramite
Ethernet ad un dispositivo in grado di fornire un indirizzo IP dinamico tramite
DHCP, ed in seguito ad instradare i pacchetti dal Finder Opta ad Internet e
viceversa.

### Panoramica del codice

Lo scopo del seguente tutorial è di configurare le impostazioni di rete di
Finder Opta, prima staticamente e poi dinamicamente. In questa maniera Finder
Opta sarà connesso ad Internet, e protremo quindi eseguire dei check di
connettività.

Il codice completo di esempio con indirizzo IP statico è disponibile
[qui](assets/OptaEthernetStatic.zip), mentre la versione utilizzante il DHCP si
trova a [questo link](assets/OptaEthernetDHCP.zip). Dopo aver estratto i file,
gli sketch possono essere compilati e caricati sul Finder Opta.

#### Configurazione con indirizzo IP statico

In questo caso sarà necessario configurare il Finder Opta con un indirizzo IP
statico appartenente alla stessa LAN del router, oltre che definire l'indirizzo
IP del server DNS che si intende utilizzare. Una volta definite le variabili,
basterà accedere al MAC address del dispositivo ed utilizzare la funzione
`begin()` della classe `Ethernet`.

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

In caso lo sketch non riesca a leggere il MAC address del dispositivo segnalerà
il problema facendo accendere il LED 1. Se tutto va come previsto il nostro
Finder Opta disporrà di un indirizzo IP e di un server DNS, e potremo procedere
con l'esecuzione del programma.

#### Configurazione con indirizzo IP dinamico

In questo caso basterà fornire il MAC address del Finder Opta alla funzione
`begin()` che si occuperà di ottenere indirizzo IP dinamico e indirizzo IP del
server DNS tramite DHCP:

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

In caso lo sketch non riesca a leggere il MAC address del dispositivo segnalerà
il problema facendo accendere il LED 1. In caso di problemi durante il lease
DHCP sarà il LED 0 ad accendersi. Se invece non ci saranno errori, il nostro
Finder Opta disporrà di un indirizzo IP e di un server DNS, e potremo procedere
con l'esecuzione del programma.

#### Check di connettività

Di seguito troviamo il codice della funzione `loop()`:

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

Questa funzione effettua un check di connettività verso la porta 80 del server
specificato nello sketch, ed accende il LED 3 in caso di successo o il LED 2 in
caso di problemi.

## Conclusioni

Questo tutorial mostra come assegnare un indirizzo IP statico o dinamico a
Finder Opta, effettuando  poi un check di connettività per verificare che il
dispositivo possa comunicare con un server remoto tramite Ethernet.
