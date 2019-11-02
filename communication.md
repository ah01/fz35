# XY-FZ25/35 Communication Description

<img src="https://img.shields.io/badge/Status-Work_In_Progress-yellow.svg" alt="status: in progress">

## Interface

It is *serial communication* with following parameters:

### Parameters

<table>
<tr><td>Baud Rate</td><td> 9600 bps</td></tr>
<tr><td> Data bits </td><td>  8 </td></tr>
<tr><td> Stop bits </td><td>  1 </td></tr>
<tr><td> Parity </td><td>  None </td></tr>
<tr><td> Flow control </td><td>  None </td></tr>
</table>

### Electric characteristics

- TTL level communication (5V level, there is XL1509-5.0 buck voltage regulator)

:exclamation: **Warning**: TX and RX pins are connected directly to MCU pins without any protection.

:exclamation: **Warning**: There is **NO** galvanic isolation between communication interface, power supply or load input.

## Protocol

- Master - slave communication
- Commands are send **without** any line ending
- Reply are ended with `CRLF`

### Commands

| Command | Reply | Note |
|---------------|------|------|
| `start` | S/F | Start periodic measurement upload |
| `stop` | S/F | Stop upload |
| `on` | S/F | Turn on Load features |
| `off` | S/F | Turn off Load features |
| `x.xxA` | S/F | Set load current |
| `LVP:xx.x` | S/F | Set Low Voltage Protection |
| `OVP:xx.x` | S/F | Set Over Voltage Protection |
| `OCP:x.xx` | S/F | Set Over Current Protection |
| `OPP:xx.xx` | S/F | Set Over Power Protection |
| `OAH:x.xxx` | S/F | Set maximum capacity |
| `OHP:xx:xx` | S/F | Set maximum discharge time |
| `read` | parameters | Read product parameter settings |

### S/F Replays (success/fail)

Most commands have reply just `success` or `fail`.

Fail usualy mean wrong format or value out of range. Especially format is tricky - you need to send exact same decimal places as required (including leading and ending zeros) and also **no line ending**!

Examples: `OPP:05.00` will work, but `OPP:5` or `OPP:5.0` or `OPP:05.00<CR><LF>` will not.

> :bulb: Note: My FZ35 return in success case `sucess` (sic). But I saw some implementation of communication library that expected `success` with correct spelling. So there is maybe more FW versions out there.

### Parameters reply

For `read` command device reply with current setting in following format:

```
OVP:xx.x, OCP:x.xx, OPP:xx.xx, LVP:xx.x,OAH:x.xxx,OHP:xx:xx<CR><LF>
```

> Note: Spaces are correct.

### Measurement upload

After `start` command device will start sending current measurement every 1 sec. Format:

```
xx.xxV,x.xA,x.xxxAh,xx:xx<CR><LF>
```

One line contains 4 values:

- voltage
- current
- capacity 
- remaining time (if OHP is set, otherwise `00:00`)

If output is OFF all values are 0.

If output is ON and there is nothing connected, current value is set value.

**TODO** alarm states

## End notes

- all settings (including state and measurement upload) is persistent.
