Note: This repository has been archived and no longer maintained. Please check [QEMU project's web site](https://www.qemu.org/). Thank you.

# nvme-model-for-qemu

 This is an implementation of a drive model for Qemu that is (almost) compliant to NVMe specification

# Notes

* based on qemu-4.1.0
* Functions of some features are limited to be able to check only request and response between host and storage

# Current Implementation

It is referred to the NVMe 1.3.0 specification.

## Controller Register
### Offset 00h: Controller Capabilities (CAP)

|   Bit | Mnemonic | Value        | Note                                            |
|------:|:---------|:-------------|:------------------------------------------------|
| 15:00 | MQES     | 0x7FF (2047) | means 2048 (0's based)                          |
|    16 | CQR      | 1            | queues are required to be physically contiguous |
| 18:17 | AMS      | 0            | only round robin arbitration mechanism          |
| 23:19 |          |              | _reserved_                                      |
| 31:24 | TO       | 0xF (15)     | 15 x 500msec = 7500msec = 7.5sec                |
| 35:32 | DSTRD    | 0            | 2 ^ (2 + 0) = 4 bytes                           |
|    36 | NSSRS    | 0            | not supported                                   |
| 44:37 | CSS      | 1            | supports NVM command set                        |
|    45 | BPS      | 0            | not supported                                   |
| 47:46 |          |              | _reserved_                                      |
| 51:48 | MPSMIN   | 0            | 2 ^ (12 + 0) =  4096 bytes =  4 KiB             |
| 55:52 | MPSMAX   | 4            | 2 ^ (12 + 4) = 65536 bytes = 64 KiB             |
| 63:56 |          |              | _reserved_                                      |

### Offset 08h: Version (VS)

|   Bit | Mnemonic | Value | Note                   |
|------:|:---------|:------|:-----------------------|
| 07:00 | TER      | 0     |                        |
| 15:08 | MER      | 3     |                        |
| 31:16 | MJR      | 1     |                        |

## Identify Controller data structure

|      Byte | O/M | Mnemonic | Value            | Note                         |
|----------:|:---:|:---------|:-----------------|:-----------------------------|
|   01:  00 | M   | VID      | --               | environment dependent        |
|   03:  02 | M   | SSVID    | --               | environment dependent        |
|   23:  04 | M   | SN       | --               | environment dependent        |
|   63:  24 | M   | MN       | "QEMU NVMe Ctrl" |                              |
|   71:  64 | M   | FR       | "1.0"            |                              |
|        72 | M   | RAB      | 6                | means 64 (2^6)               |
|   75:  73 | M   | IEEE     | 0x0002B3         |                              |
|        76 | O   | CMIC     | 0                |                              |
|        77 | M   | MDTS     | 0                | means 1 (2^0)                |
|   79:  78 | M   | CNTLID   | 0                |                              |
|   83:  80 | M   | VER      | 0x00010200       | means 1.2.0                  |
|   87:  84 | M   | RTD3R    | 0                |                              |
|   91:  88 | M   | RTD3E    | 0                |                              |
|   95:  92 | M   | OAES     | 0                |                              |
|  239:  96 |     |          |                  | _reserved_                   |
|  255: 240 |     |          |                  | _Refer to the NVMe-MI Spec._ |
|  257: 256 | M   | OACS     | 0                |                              |
|       258 | M   | ACL      | 0                | means 1 (0's based value)    |
|       259 | M   | AERL     | 0                | means 1 (0's based value)    |
|       260 | M   | FRMW     | 0x0E             | there's no activation pending slot, active slot is slot 1, and number of FW slot is seven |
|       261 | M   | LPA      | 0xA              | supports "Command Supported and Effects" and Telemetry |
|       262 | M   | ELPE     | 0                | means 1 (0's based value)    |
|       263 | M   | NPSS     | 0                | means 1 (2^0)                |
|       264 | M   | AVSCC    | 0                |                              |
|       265 | O   | APSTA    | 0                |                              |
|  267: 266 | M   | WCTEMP   | 0                |                              |
|  269: 268 | M   | CCTEMP   | 0                |                              |
|  271: 270 | O   | MTFA     | 0                |                              |
|  275: 272 | O   | HMPRE    | 0                |                              |
|  279: 276 | O   | HMMIN    | 0                |                              |
|  295: 280 | O   | TNVMCAP  | 0                |                              |
|  311: 296 | O   | UNVMCAP  | 0                |                              |
|  315: 312 | O   | RPMBS    | 0                |                              |
|  511: 316 |     |          |                  | _reserved_                   |
|       512 | M   | SQES     | 0x66             | 64 bytes                     |
|       513 | M   | CQES     | 0x44             | 16 bytes                     |
|  515: 514 |     |          |                  | _reserved_                   |
|  519: 516 | M   | NN       | --               | environment dependent        |
|  521: 520 | M   | ONCS     | 0x4C             | supports Write Zeroes and Dataset Management (only Deallocate) |
|  523: 522 | M   | FUSES    | 0                |                              |
|       524 | M   | FNA      | 0                |                              |
|       525 | M   | VWC      | --               | environment dependent        |
|  527: 526 | M   | AWUN     | 0                |                              |
|  529: 528 | M   | AWUPF    | 0                |                              |
|       530 | M   | NVSCC    | 0                |                              |
|       531 |     |          |                  | _reserved_                   |
|  533: 532 | O   | ACWU     | 0                |                              |
|  535: 534 |     |          |                  | _reserved_                   |
|  539: 536 | O   | SGLS     | 0                | SGL is not supported         |
|  703: 540 |     |          |                  | _reserved_                   |
| 2047: 704 |     |          |                  | _reserved_                   |
| 2079:2048 | M   | PSD0     | _see below_      |                              |
| 3071:2080 | O   | PSD1-31  |                  |                              |
| 4095:3072 |     |          |                  | _Vendor Specific_            |

## Power State 0 Descriptor (PSD0)

|     Bit | Mnemonic | Value | Note                  |
|--------:|:---------|:------|:----------------------|
|  15: 00 | MP       | 0x9C4 |                       |
|  23: 16 |          |       | _reserved_            |
|      24 | MPS      | 0     |                       |
|      25 | NOPS     | 0     |                       |
|  31: 26 |          |       | _reserved_            |
|  63: 32 | ENLAT    | 0x10  | 10 microseconds       |
|  95: 64 | EXLAT    | 0x4   | 4 microseconds        |
| 100: 96 | RRT      | 0     |                       |
| 103:101 |          |       | _reserved_            |
| 108:104 | RRL      | 0     |                       |
| 111:109 |          |       | _reserved_            |
| 116:112 | RWT      | 0     |                       |
| 119:117 |          |       | _reserved_            |
| 124:120 | RWL      | 0     |                       |
| 127:125 |          |       | _reserved_            |
| 143:128 | IDLP     | 0     |                       |
| 149:144 |          |       | _reserved_            |
| 151:150 | IPS      | 0     |                       |
| 159:152 |          |       | _reserved_            |
| 175:160 | ACTP     | 0     |                       |
| 178:176 | APW      | 0     |                       |
| 181:179 |          |       | _reserved_            |
| 183:182 | APS      |       |                       |
| 255:184 |          |       | _reserved_            |

## Identify Namespace data structure for Namespace 1

|      Byte | O/M | Mnemonic | Value     | Note                      |
|----------:|:---:|:---------|:----------|:--------------------------|
|   07:  00 | M   | NSZE     | --        | environment dependent     |
|   15:  08 | M   | NCAP     | --        | environment dependent     |
|   23:  16 | M   | NUSE     | --        | environment dependent     |
|        24 | M   | NSFEAT   | 0         |                           |
|        25 | M   | NLBAF    | 0         |                           |
|        26 | M   | FLBAS    | 0         |                           |
|        27 | M   | MC       | 0         |                           |
|        28 | M   | DPC      | 0         |                           |
|        29 | M   | DPS      | 0         |                           |
|        30 | O   | NMIC     | 0         |                           |
|        31 | O   | RESCAP   | 0         |                           |
|        32 | O   | FPI      | 0         |                           |
|        33 | O   | DLFEAT   | 0         |                           |
|   35:  34 | O   | NAWUN    | 0         |                           |
|   37:  36 | O   | NAWUPF   | 0         |                           |
|   39:  38 | O   | NACWU    | 0         |                           |
|   41:  40 | O   | NABSN    | 0         |                           |
|   43:  42 | O   | NABO     | 0         |                           |
|   45:  44 | O   | NABSPF   | 0         |                           |
|   47:  46 | O   | NOIOB    | 0         |                           |
|   63:  48 | O   | NMVCAP   | 0         |                           |
|  103:  64 | O   |          |           | _reserved_                |
|  119: 104 | O   | NGUID    | 0         |                           |
|  127: 120 | O   | EUI64    | 0         |                           |
|  131: 128 | M   | LBAF0    | _see note._ | RP = 0, LBADS = 9, MS = 0 |
|  191: 132 | O   | LBAF1-15 |           |                           |
|  383: 192 |     |          |           | _reserved_                |
| 4095: 384 |     |          |           | _Vendor Specific_         |

## SMART / Health Information
It can be retreieved by "Get Log Page" command with Log Identifier 01h.

| Byte     | Value | Description                             | Note               |
|---------:|------:|:----------------------------------------|:------------------:|
|        0 |     0 | Critical Warning                        ||
|   2:   1 |   305 | Composite Temperature                   | 30 degrees Celsius |
|        3 |   100 | Available Spare                         ||
|        4 |    10 | Available Spare Threshold               ||
|        5 |     0 | Percentage Used                         ||
|  31:   6 |       | _reserved_                              ||
|  47:  32 |     0 | Data Units Read                         ||
|  63:  48 |     0 | Data Units Written                      ||
|  79:  64 |     0 | Host Read Commands                      ||
|  95:  80 |     0 | Host Write Commands                     ||
| 111:  96 |     0 | Controller Busy Time                    ||
| 127: 112 |       | Power Cycles                            ||
| 143: 128 |     0 | Power On Hours                          ||
| 159: 144 |     0 | Unsafe Shutdowns                        ||
| 175: 160 |     0 | Media and Data Integrity Errors         ||
| 191: 176 |     0 | Number of Error Information Log Entries ||
| 195: 192 |     0 | Warning Composite Temperature Time      ||
| 199: 196 |     0 | Critical Composite Temperature Time     ||
| 201: 200 |   305 | Temperature Sensor 1                    | 30 degrees Celsius |
| 203: 202 |     0 | Temperature Sensor 2                    ||
| 205: 204 |     0 | Temperature Sensor 3                    ||
| 207: 206 |     0 | Temperature Sensor 4                    ||
| 209: 208 |     0 | Temperature Sensor 5                    ||
| 211: 210 |     0 | Temperature Sensor 6                    ||
| 213: 212 |     0 | Temperature Sensor 7                    ||
| 215: 214 |     0 | Temperature Sensor 8                    ||
| 511: 216 |       | _reserved_                              ||

## Command Support

It can be retreieved by "Get Log Page" command with Log Identifier 05h (Commands Supported and Effects log page).

### Admin Command

| Opecode  | Description                 | Support           | Note              |
|---------:|:------|:----------------------------------------|:------------------|
|      00h | Delete I/O Submission Queue | x                 ||
|      01h | Create I/O Submission Queue | x                 ||
|      02h | Get Log Page                | x                 ||
|      04h | Delete I/O Completion Queue | x                 ||
|      05h | Create I/O Completion Queue | x                 ||
|      06h | Identify                    | x                 ||
|      08h | Abort                       |                   ||
|      09h | Set Features                | x                 ||
|      0Ah | Get Features                | x                 ||
|      0Ch | Asynchronous Event Request  |                   ||
|      0Dh | Namespace Management        |                   ||
|      10h | Firmware Commit             |                   ||
|      11h | Firmware Image Download     |                   ||
|      15h | Namespace Attachment        |                   ||
|      80h | Format NVM                  |                   ||
|      81h | Security Send               |                   ||
|      82h | Security Receive            |                   ||

### I/O Command

| Opecode  | Description                 | Support           | Note              |
|---------:|:------|:----------------------------------------|:------------------|
|      00h | Flush                       | x                 ||
|      01h | Write                       | x                 ||
|      02h | Read                        | x                 ||
|      04h | Write Uncorrectable         |                   ||
|      05h | Compare                     |                   ||
|      08h | Write Zeroes                | x                 ||
|      09h | Dataset Management          | x                 | `Deallocate` only |
|      0Dh | Reservation Register        |                   ||
|      0Eh | Reservation Report          |                   ||
|      0Dh | Namespace Management        |                   ||
|      11h | Reservation Acquire         |                   ||
|      15h | Reservation Release         |                   ||

## Log Page Support

| Log Id   | Description                 | Support           | Note              |
|---------:|:------|:----------------------------------------|:------------------|
|      01h | Error Information           | x                 | No error Information is created. |
|      02h | SMART / Health Information  | x                 ||
|      03h | Firmware Slot Information   | x                 ||
|      04h | Changed Namespace List      |                   ||
|      05h | Commands Supported and Effects | x              ||
|      06h | Device Self-test            |                   ||
|      07h | Telemetry Host-Initiated    | x                 | No telemetry data is created; only header is returned. See also Note 1. |
|      08h | Telemetry Controller-Initiated | x              | No telemetry data is created; only header is returned. |
|      70h | Discovery                   |                   ||
|      80h | Reservation Notification    |                   ||
|      0Dh | Sanitize Status             |                   ||

Note 1: if "Create Telemetry Host-Initiated Data" is set to `1`, the data format of the response is not followed to NVMe spec because Windows 10 requests different data format...
