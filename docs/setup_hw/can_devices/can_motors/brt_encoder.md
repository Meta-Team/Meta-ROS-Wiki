# BRITER Encoders

[BRITER](https://www.buruiter.com/) is an encoder manufacturer. Some of our motors lacking absolute zero (e.g. MI CyberGear) can be used with BRITER encoders to achieve absolute zero.

## Product Manuals
[click here](https://www.buruiter.com/wp-content/uploads/2023/08/003-CAN%E8%AF%B4%E6%98%8E%E4%B9%A6%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE%E5%8D%95%E5%9C%88V2.2.pdf)

## Packet Structure

```
0x001 # [LEN] [ID] [FUNC] [DATA]
```

## Useful Commands

Read encoder value from ID 1:

```bash
cansend can0 001#04010100
```

Set encoder ID 1 to 2:

```bash
cansend can0 001#04010202
```

Set bitrate of ID 1 to 1Mbps (remember to always set the bitrate to 1Mbps, all of our motors run at 1Mbps):

```bash
cansend can0 001#04010301
```
Set encoder ID 1 to automatic feedback mode:

```bash
cansend can0 001#040104AA
```

Set ID 1 automatic feedback interval to 1000ms:

```bash
cansend can0 001#050105E803
```

Set current position as zero (for ID 1):

```bash
cansend can0 001#04010600
```
