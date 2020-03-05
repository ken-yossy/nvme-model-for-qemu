# nvme-model-for-qemu

 This is a personal implementation of a drive model for Qemu that is compliant to NVMe specification

# Notes

* based on qemu-4.1.0

# Current Implementation (referred to the NVMe 1.3d specification)

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
| 15:08 | MER      | 2     | will be changed to '3' |
| 31:16 | MJR      | 1     |                        |

## Identify Controller data structure

|      Byte | O/M | Mnemonic | Exists? | Value            | Note                         |
|----------:|:---:|:---------|:-------:|:-----------------|:-----------------------------|
|   01:  00 | M   | VID      | x       | --               | environment dependent        |
|   03:  02 | M   | SSVID    | x       | --               | environment dependent        |
|   23:  04 | M   | SN       | x       | --               | environment dependent        |
|   63:  24 | M   | MN       | x       | "QEMU NVMe Ctrl" |                              |
|   71:  64 | M   | FR       | x       | "1.0"            |                              |
|        72 | M   | RAB      | x       | 6                | means 64 (2^6)               |
|   75:  73 | M   | IEEE     | x       | 0x0002B3         |                              |
|        76 | O   | CMIC     | x       | 0                |                              |
|        77 | M   | MDTS     | x       | 0                | means 1 (2^0)                |
|   79:  78 | M   | CNTLID   |         |                  |                              |
|   83:  80 | M   | VER      |         |                  |                              |
|   87:  84 | M   | RTD3R    |         |                  |                              |
|   91:  88 | M   | RTD3E    |         |                  |                              |
|   95:  92 | M   | OAES     |         |                  |                              |
|   99:  96 | M   | CTRATT   |         |                  |                              |
|  111: 100 |     |          |         |                  | _reserved_                   |
|  127: 112 | O   | FGUID    |         |                  |                              |
|  239: 128 |     |          |         |                  | _reserved_                   |
|  255: 240 |     |          |         |                  | _Refer to the NVMe-MI Spec._ |
|  257: 256 | M   | OACS     | x       | 0                |                              |
|       258 | M   | ACL      | x       | 0                | means 1 (0's based value)    |
|       259 | M   | AERL     | x       | 0                | means 1 (0's based value)    |
|       260 | M   | FRMW     | x       | 0x0E             |                              |
|       261 | M   | LPA      | x       | 0x1              |                              |
|       262 | M   | ELPE     | x       | 0                | means 1 (0's based value)    |
|       263 | M   | NPSS     | x       | 0                | means 1 (2^0)                |
|       264 | M   | AVSCC    |         |                  |                              |
|       265 | O   | APSTA    |         |                  |                              |
|  267: 266 | M   | WCTEMP   |         |                  |                              |
|  269: 268 | M   | CCTEMP   |         |                  |                              |
|  271: 270 | O   | MTFA     |         |                  |                              |
|  275: 272 | O   | HMPRE    |         |                  |                              |
|  279: 276 | O   | HMMIN    |         |                  |                              |
|  295: 280 | O   | TNVMCAP  |         |                  |                              |
|  311: 296 | O   | UNVMCAP  |         |                  |                              |
|  315: 312 | O   | RPMBS    |         |                  |                              |
|  317: 316 | O   | EDSTT    |         |                  |                              |
|       318 | O   | DSTO     |         |                  |                              |
|       319 | M   | FWUG     |         |                  |                              |
|  321: 320 | M   | KAS      |         |                  |                              |
|  323: 322 | O   | HCTMA    |         |                  |                              |
|  325: 324 | O   | MNTMT    |         |                  |                              |
|  327: 326 | O   | MXTMT    |         |                  |                              |
|  331: 328 | O   | SANICAP  |         |                  |                              |
|  511: 332 |     |          |         |                  | _reserved_                   |
|       512 | M   | SQES     | x       | 0x66             | 64 bytes                     |
|       513 | M   | CQES     | x       | 0x44             | 16 bytes                     |
|  515: 514 | M   | MAXCMD   |         |                  |                              |
|  519: 516 | M   | NN       | x       | --               | environment dependent        |
|  521: 520 | M   | ONCS     | x       | 0x48             |                              |
|  523: 522 | M   | FUSES    | x       | 0                |                              |
|       524 | M   | FNA      | x       | 0                |                              |
|       525 | M   | VWC      | x       | --               | environment dependent        |
|  527: 526 | M   | AWUN     | x       | 0                |                              |
|  529: 528 | M   | AWUPF    | x       | 0                |                              |
|       530 | M   | NVSCC    |         |                  |                              |
|       531 |     |          |         |                  | _reserved_                   |
|  533: 532 | O   | ACWU     |         |                  |                              |
|  535: 534 |     |          |         |                  | _reserved_                   |
|  539: 536 | O   | SGLS     |         |                  |                              |
|  767: 540 |     |          |         |                  | _reserved_                   |
| 1023: 768 | M   | SUBNQN   |         |                  |                              |
| 1791:1024 |     |          |         |                  | _reserved_                   |
| 2047:1792 |     |          |         |                  | _Refer to the NVMeoF Spec._  |
| 2079:2048 | M   | PSD0     | x       |                  |                              |
| 3071:2080 | O   | PSD1-31  | x       |                  |                              |
| 4095:3072 |     |          |         |                  | _Vendor Specific_            |

## Power State 0 Descriptor (PSD0)

|     Bit | Mnemonic | Exists? | Value | Note                  |
|--------:|:---------|:-------:|:------|:----------------------|
|  15: 00 | MP       | x       | 0x9C4 | environment dependent |
|  23: 16 |          |         |       | _reserved_            |
|      24 | MXPS     |         |       |                       |
|      25 | NOPS     |         |       |                       |
|  31: 26 |          |         |       | _reserved_            |
|  63: 32 | ENLAT    | x       | 0x10  | 10 microseconds       |
|  95: 64 | EXLAT    | x       | 0x4   | 4 microseconds        |
| 100: 96 | RRT      | x       | 0     |                       |
| 103:101 |          |         |       | _reserved_            |
| 108:104 | RRL      | x       | 0     |                       |
| 111:109 |          |         |       | _reserved_            |
| 116:112 | RWT      | x       | 0     |                       |
| 119:117 |          |         |       | _reserved_            |
| 124:120 | RWL      | x       | 0     |                       |
| 127:125 |          |         |       | _reserved_            |
| 143:128 | IDLP     |         |       |                       |
| 149:144 |          |         |       | _reserved_            |
| 151:150 | IPS      |         |       |                       |
| 159:152 |          |         |       | _reserved_            |
| 175:160 | ACTP     |         |       |                       |
| 178:176 | APW      |         |       |                       |
| 181:179 |          |         |       | _reserved_            |
| 183:182 | APS      |         |       |                       |
| 255:184 |          |         |       | _reserved_            |

## Identify Namespace data structure for Namespace 1

|      Byte | O/M | Mnemonic | Exists? | Value     | Note                      |
|----------:|:---:|:---------|:-------:|:----------|:--------------------------|
|   07:  00 | M   | NSZE     | x       | --        | environment dependent     |
|   15:  08 | M   | NCAP     | x       | --        | environment dependent     |
|   23:  16 | M   | NUSE     | x       | --        | environment dependent     |
|        24 | M   | NSFEAT   | x       | 0         |                           |
|        25 | M   | NLBAF    | x       | 0         |                           |
|        26 | M   | FLBAS    | x       | 0         |                           |
|        27 | M   | MC       | x       | 0         |                           |
|        28 | M   | DPC      | x       | 0         |                           |
|        29 | M   | DPS      | x       | 0         |                           |
|        30 | O   | NMIC     |         |           |                           |
|        31 | O   | RESCAP   |         |           |                           |
|        32 | O   | FPI      |         |           |                           |
|        33 | O   | DLFEAT   |         |           |                           |
|   35:  34 | O   | NAWUN    |         |           |                           |
|   37:  36 | O   | NAWUPF   |         |           |                           |
|   39:  38 | O   | NACWU    |         |           |                           |
|   41:  40 | O   | NABSN    |         |           |                           |
|   43:  42 | O   | NABO     |         |           |                           |
|   45:  44 | O   | NABSPF   |         |           |                           |
|   47:  46 | O   | NOIOB    |         |           |                           |
|   63:  48 | O   | NMVCAP   |         |           |                           |
|  103:  64 | O   |          |         |           | _reserved_                |
|  119: 104 | O   | NGUID    |         |           |                           |
|  127: 120 | O   | EUI64    |         |           |                           |
|  131: 128 | M   | LBAF0    | x       | see note. | RP = 0, LBADS = 9, MS = 0 |
|  191: 132 | O   | LBAF1-15 | x       |           |                           |
|  383: 192 |     |          |         |           | _reserved_                |
| 4095: 384 |     |          |         |           | _Vendor Specific_         |
