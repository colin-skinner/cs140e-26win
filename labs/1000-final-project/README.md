# Final Project - Colin Skinner

My final project for CS140E winter quarter was to get a demo with a network thread running and communicating over ICMP. I used SPI to communicate with a MAC/PHY: the [W5500](https://wiznet.io/products/ethernet-chips/w5500) breakout ([Link on amazon](https://www.amazon.com/ACEIRMC-Ethernet-Network-Interface-WIZ820io/dp/B0DTTLGJCS/ref=sr_1_4?crid=14LAKUF8FZFNZ&dib=eyJ2IjoiMSJ9.gRnxM8HDGZWO4dMiRqyegzF-gMkuasnUxKFIHX1Qa_XlCBqzKTWzekT26aF_jvChMSJ6SzZGmT4skXVGUNcBX4iAHj6GfASuv5lXBYJBgZ6g1gUFg8IBfYouOTl1GqT-vBscXUrYMEXHMUA-q0VrwhiWmebpJBoGUElUjU2wJN1PoRbynJmnrUIw4BuexWysK7Dm7kL8cEmhKDGJ1YKwm74pE5l71ZQd9FzZmY3XZPI.kAj-B7cdMY-nHFg3-zRZ5WYELcatoHc18K1v9TWXlos&dib_tag=se&keywords=w5500&qid=1774046861&refresh=1&sprefix=w5%2Caps%2C383&sr=8-4&th=1)). I was originally going to use the onboard Broadcom WiFi chip, but chose the W5500 for several reasons:
1. Broadcom's chip is proprietary so there is no publicly-available datasheet
2. I already had the W5500 and have experience in SPI and creating hardware sensor/peripheral drivers 
3. The W5500 uses Ethernet, and wired connections are more representative of what I would be using TCP/IP-like networking for on a spacecraft (as I am studying AeroAstro)
This README is a reflection on the project and how I went about designing it:

## Order of development
First off, I used [Wireshark](https://www.wireshark.org/) to debug all of my Ethernet frames put on the wired connection I had. This was immeasurably useful, and I wouldn't have been able to complete this much without it
1. **W5500 Driver**
   1. I started first with using the SPI functions to read the Chip ID and mimic the put_chk interface of the NRF radio
   2. Then I sifted through the [datasheet](https://cdn.sparkfun.com/datasheets/Dev/Arduino/Shields/W5500_datasheet_v1.0.2_1.pdf) to find which values needed to be written to the registers for the MACRAW mode (with raw Ethernet frames) to be enabled
      1. The chip is also capable of up to Layer 4 networking, holding UDP and TCP sockets, but I opted to learn from layer 2 (for the love of the game)
   3. I ended up troubleshooting the TX and RX buffers for a while because of the buffer pointer registers wrapping around the 2KB automatically when I didn't expect them to, so I was double-checking the pointer logic and making it super slow (I fixed this by looking at the W5500 official C code driver and realizing how overengineered, but legible, my code is)
2. **Ethernet**
   1. With the important (and at the start, hasty) assumption that Layer 1-ish is done, I went on to Ethernet
      1. (it was hasty because again, I had to fix the buffer pointer wrapping issue several times)
   2. At first, I just used the Broadcast MAC (`FF:FF:FF:FF:FF:FF`) but after I got ARP to work, it was easier
   3. Important note: Endianness:
      1. Variables in bitfields are **not** guaranteed to be in order. This was evident during the version/header length part of the Ethernet frame
      2. Internet is generally big-endian. So that's tough. Gotta swap things around OR do it in byte order. The smarter thing would have been to separate the "API struct" from the "raw data struct/buffer" to guarantee order of the bytes/nibbles
3. **ICMP**
4. **IPV4**
5. **UDP**


## Scrapbook/Brainstorm
### Useful links

- [IP Numbers](https://www.iana.org/assignments/protocol-numbers/protocol-numbers.xhtml)
- [FTP RFC 114](https://www.rfc-editor.org/rfc/rfc114.txt)
- [UDP RFC 768](https://www.rfc-editor.org/rfc/rfc768)
- [ICMP RFC 792](https://datatracker.ietf.org/doc/html/rfc792)
### Plan
-  [ ] Frames
   -  [ ] More filtering
      -  [ ] EtherType in higher level handler
      -  [ ] Length
-  [ ] ARP [RFC 826](https://www.rfc-editor.org/rfc/rfc826)
   -  already have FRAME_ARP
   -  Put into frame handler
-  [ ] IPv4
   -  [ ] More filtering
-  [ ] UDP
-  [ ] 
-  [ ] TCP
-  [ ] FTP
  

1. Ethernet frame filtering
2. Implement ARP table + ARP reply
3. Refactor IPv4 parsing into its own module
4. IPv4 checksum
5. UDP with ports
6. TCP
7. Build application protocols 
   1. FTP on TCP

## Working
- ARP
- IPv4
- UDP / ICMP



## Adding new protocol
- Change lower level protocol handler
- Add verbosity
- Add init to inet_init


- Had to edit rpi_yield to make sure it is only yielding if threads are enabled
- put structs statically in their files (like udp_t)