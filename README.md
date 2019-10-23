# GREat LoRa

```Alexandre Barroso Costa - alexandrebc00@gmail.com - @alexandrebc (tg)```

## Overview
Rede LoRa
![lora](https://www.thethingsnetwork.org/docs/lorawan/LoRaWAN-Overview.png)
        - 
## Gateway
- Geral
  -  [Manual](https://www.dragino.com/downloads/downloads/UserManual/LG01_LoRa_Gateway_User_Manual.pdf)
  -  [Wiki - Connect to TTN](https://wiki.dragino.com/index.php?title=Connect_to_TTN)
  -  Dragino
  -  LG01-P
  -  ssh root@10.130.1.1 - dragino
- AP
  - dragino-1ca494
  - GRT@dragino2019
- LoRaWAN Server Settings
  - [Server Address] router.au.thethings.network : 1700
  - [Gateway ID] a840411ca494ffff
  - [RX/TX Frequency] 916800000
  - [Spreading Factor] SF7
  - [Transmit Spreading Factor] SF9
  - [Coding Rate] 4/5
  - [Signal Bandwith] 125 kHz
  - [Preamble Lenght] 8

## Node
- Geral
    - [TTGO ESP32 LoRa](https://github.com/osresearch/esp32-ttgo)
    - Pinagem: ![Pinagem](https://github.com/osresearch/esp32-ttgo/raw/master/images/esp32-pinout2.jpg)

### Arduino
- Placa
    - Arquivo > Preferências > URL Adicionais
        - https://dl.espressif.com/dl/package_esp32_index.json
    - Ferramentas > Placa > Gerenciador de Placa
        - "esp"
        - [Configurações para programar](http://prntscr.com/p7fuq5)
- Bibliotecas
    - [OLED](https://github.com/ThingPulse/esp8266-oled-ssd1306)
    - [LoRa](https://github.com/sandeepmistry/arduino-LoRa)
    - [LoRaWaN](LoRa-GREat-master.zip)
        - Biblioteca modificada para funcionamento com as configurações utilizadas no Dragino LG01-P do GREat
        - Para utilizar a biblioteca com a TTGO a pinagem é a seguinte
        - [Referência](https://github.com/thomaslaurenson/Dragino_LoRaShield_Node_AU915)
        - 
        ```c 
            const lmic_pinmap lmic_pins = {
                .nss = 18,
                .rxtx = LMIC_UNUSED_PIN,
                .rst = 14,
                .dio = {26, 33, 32},
            };
        ```

## LoRa
### Informações Gerais
- Distância Alcancada
  - transmissao continua até 180m
  - 1 pacote a 500m

### Australian and New Zealand Frequencies
| Channel 	| Direction 	| Frequency (MHz) 	| Bandwidth (kHz) 	|  Data Rate 	|
|:-------:	|:---------:	|:---------------:	|:---------------:	|:----------:	|
|    8*    	|     up    	|      916.8      	|       125       	|  DR0 - DR3 	|
|    9    	|     up    	|      917.0      	|       125       	|  DR0 - DR3 	|
|    10   	|     up    	|      917.2      	|       125       	|  DR0 - DR3 	|
|    11   	|     up    	|      917.4      	|       125       	|  DR0 - DR3 	|
|    12   	|     up    	|      917.6      	|       125       	|  DR0 - DR3 	|
|    13   	|     up    	|      917.8      	|       125       	|  DR0 - DR3 	|
|    14   	|     up    	|      918.0      	|       125       	|  DR0 - DR3 	|
|    15   	|     up    	|      918.2      	|       125       	|  DR0 - DR3 	|
|    65   	|     up    	|      917.5      	|       500       	|     DR4    	|
|    0    	|    down   	|      923.3      	|       500       	| DR8 - DR13 	|
|    1    	|    down   	|      923.9      	|       500       	| DR8 - DR13 	|
|    2    	|    down   	|      924.5      	|       500       	| DR8 - DR13 	|
|    3    	|    down   	|      925.1      	|       500       	| DR8 - DR13 	|
|    4    	|    down   	|      925.7      	|       500       	| DR8 - DR13 	|
|    5    	|    down   	|      926.3      	|       500       	| DR8 - DR13 	|
|    6    	|    down   	|      926.9      	|       500       	| DR8 - DR13 	|
|    7    	|    down   	|      927.5      	|       500       	| DR8 - DR13 	|

Obs.: O projeto utiliza o canal 8 (banda 916.8E6) para o tráfego da informação.

## Código teste utilizando OTAA para TTN
```c
/*******************************************************************************
 * Specific modifications for use of LoRa ESP32 TTGO on AU915, sub-band 2
 * This example sends a valid LoRaWAN packet with payload "Hello,
 * world!", using frequency and encryption settings matching those of
 * the (early prototype version of) The Things Network.
 *
 * Change DEVADDR to a unique address!
 * See http://thethingsnetwork.org/wiki/AddressSpace
 *
 * Do not forget to define the radio type correctly in config.h.
 *******************************************************************************/

#include <lmic.h>
#include <hal/hal.h>
#include <SPI.h>

// LoRaWAN NwkSKey, network session key
// This is the default Semtech key, which is used by the prototype TTN
// network initially. -- MSB
static const PROGMEM u1_t NWKSKEY[16] = { 0x4E, 0xB0, 0x28, 0xEE, 0xBC, 0x86, 0x2B, 0x99, 0x9A, 0x28, 0xD7, 0xFD, 0xFB, 0xC5, 0x78, 0x7D };

// LoRaWAN AppSKey, application session key
// This is the default Semtech key, which is used by the prototype TTN
// network initially. -- MSB
static const u1_t PROGMEM APPSKEY[16] = { 0x97, 0xAA, 0x3D, 0xF1, 0x8D, 0x4E, 0xD6, 0x84, 0xE8, 0x0D, 0xE5, 0x6F, 0x6E, 0xE4, 0x22, 0xE2 };

// LoRaWAN end-device address (DevAddr)
// See http://thethingsnetwork.org/wiki/AddressSpace
static const u4_t DEVADDR = 0x260313EC ; // <-- Change this address for every node!

// These callbacks are only used in over-the-air activation, so they are
// left empty here (we cannot leave them out completely unless
// DISABLE_JOIN is set in config.h, otherwise the linker will complain).
void os_getArtEui (u1_t* buf) { }
void os_getDevEui (u1_t* buf) { }
void os_getDevKey (u1_t* buf) { }

static uint8_t mydata[] = "Hello, world!";
static osjob_t sendjob;

// Schedule TX every this many seconds (might become longer due to duty
// cycle limitations).
const unsigned TX_INTERVAL = 1;

// Pin mapping
const lmic_pinmap lmic_pins = {
    .nss = 18,
    .rxtx = LMIC_UNUSED_PIN,
    .rst = 14,
    .dio = {26, 33, 32},
};

void onEvent (ev_t ev) {
    Serial.print(os_getTime());
    Serial.print(": ");
    switch(ev) {
        case EV_SCAN_TIMEOUT:
            Serial.println(F("EV_SCAN_TIMEOUT"));
            break;
        case EV_BEACON_FOUND:
            Serial.println(F("EV_BEACON_FOUND"));
            break;
        case EV_BEACON_MISSED:
            Serial.println(F("EV_BEACON_MISSED"));
            break;
        case EV_BEACON_TRACKED:
            Serial.println(F("EV_BEACON_TRACKED"));
            break;
        case EV_JOINING:
            Serial.println(F("EV_JOINING"));
            break;
        case EV_JOINED:
            Serial.println(F("EV_JOINED"));
            break;
        case EV_RFU1:
            Serial.println(F("EV_RFU1"));
            break;
        case EV_JOIN_FAILED:
            Serial.println(F("EV_JOIN_FAILED"));
            break;
        case EV_REJOIN_FAILED:
            Serial.println(F("EV_REJOIN_FAILED"));
            break;
            break;
        case EV_TXCOMPLETE:
            Serial.println(F("EV_TXCOMPLETE (includes waiting for RX windows)"));
            if(LMIC.dataLen) {
                // data received in rx slot after tx
                Serial.print(F("Data Received: "));
                Serial.write(LMIC.frame+LMIC.dataBeg, LMIC.dataLen);
                Serial.println();
            }
            // Schedule next transmission
            os_setTimedCallback(&sendjob, os_getTime()+sec2osticks(TX_INTERVAL), do_send);
            break;
        case EV_LOST_TSYNC:
            Serial.println(F("EV_LOST_TSYNC"));
            break;
        case EV_RESET:
            Serial.println(F("EV_RESET"));
            break;
        case EV_RXCOMPLETE:
            // data received in ping slot
            Serial.println(F("EV_RXCOMPLETE"));
            break;
        case EV_LINK_DEAD:
            Serial.println(F("EV_LINK_DEAD"));
            break;
        case EV_LINK_ALIVE:
            Serial.println(F("EV_LINK_ALIVE"));
            break;
         default:
            Serial.println(F("Unknown event"));
            break;
    }
}

void do_send(osjob_t* j){
    // Check if there is not a current TX/RX job running
    if (LMIC.opmode & OP_TXRXPEND) {
        //Serial.println(F("OP_TXRXPEND, not sending"));
        Serial.print("OP_TXRXPEND, not sending; at freq: ");
        Serial.println(LMIC.freq);        
    } else {
        // Prepare upstream data transmission at the next possible time.
        LMIC_setTxData2(1, mydata, sizeof(mydata)-1, 0);
        Serial.print(F("Packet queued for freq: "));
        Serial.println(LMIC.freq);
    }
    // Next TX is scheduled after TX_COMPLETE event.
}

void setup() {
    Serial.begin(115200);
    Serial.println(F("Starting"));

    // LMIC init
    os_init();
    // Reset the MAC state. Session and pending data transfers will be discarded.
    LMIC_reset();

    // Set static session parameters. Instead of dynamically establishing a session
    // by joining the network, precomputed session parameters are be provided.
    // If not running an AVR with PROGMEM, just use the arrays directly 
    LMIC_setSession (0x1, DEVADDR, NWKSKEY, APPSKEY);

    // The frequency plan is hard-coded
    // But the band (or selected 8th channel) is configured here!
    // This is the same AU915 band as used by TTN
    
    // Disable all channels except the 8th
    for (int channel=0; channel<72; ++channel) {
      if(channel != 8) {
        LMIC_disableChannel(channel);
      }
    }

    // Disable link check validation
    LMIC_setLinkCheckMode(0);

    // Set data rate and transmit power (note: txpow seems to be ignored by the library)
    LMIC_setDrTxpow(DR_SF7,14);

    // Start job
    do_send(&sendjob);
}

void loop() {
    os_runloop_once();
}
```
