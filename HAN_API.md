# Octopus Home Pro Home Area Network (HAN) API

## Introduction

The Octave OS running on the main Home Pro device as a system, provides RESTful
APIs for getting energy consumption data inside the Octopus Home Pro SDK and
other parts of the system. This document explains how to access these APIs from
within the SDK and what fields are exposed.

## The HAN API Endpoint

In order to access any RESTful API, the most basic part of the connection
mechanism is knowing the endpoint that hosts the API or say responds to it. For
the HAN API to be accessed from the SDK the main system exposes an environment
variable **HAN_API_HOST** into the SDK environment.

For any python based applications, it can simply be retrieved as follows

```
import os

han_host = os.getenv('HAN_API_HOST')
```

## APIs

This section discusses the RESTful APIs that are available over the HAN API
endpoint, how these are invoked and elucidates the output.

These APIs are generally applicable to both electric and gas meters with a few
exceptions that are noted in relevant sections, if found.

### Meter Status

The meter status API `get_meter_status` can be used to fetch the status as
implied by the name. The following example demonstrates how it is used

```
import os
import requests

han_host = os.getenv('HAN_API_HOST')
meter_type = "elec"

# Call the get_meter_status API
response = requests.post(han_host + "/get_meter_status", json={"meter_type": meter_type})
if response.ok:
    meter_status = response.json()["meter_status"]
    print("Meter status for {} meter: {}".format(meter_type, meter_status))
else:
    print("Error calling get_meter_status API: {}".format(response.json()["Status"]))
```

this outputs data similar to

```
Meter status for elec meter: {
        "time": 1713535438,
        "status":       {
                "type": "elec",
                "mstatus":      0,
                "aconsumindicate":      0,
                "lastupdated":  766850639
        }
}
```

where

| Variable        | Description                                              |
| --------------- | -------------------------------------------------------- |
| type            | Indicates meter type, gas/elec.                          |
| supply          | Supply status (usually available for gas meters)         |
| mstatus         | Meter status                                             |
| aconsumindicate | Ambient amount of the commodity being consumed           |
| lastupdated     | UNIX timestamp indicating when the IHD received the data |

**supply**, where available, has the following meaning

| Value        | Description             |
| ------------ | ----------------------- |
| 0x00         | Supply OFF              |
| 0x01         | Supply OFF/ARMED        |
| 0x02         | Supply ON               |
| 0x03 to 0xFF | Reserved for future use |

**mstatus**, is a bitmap and each bit represents different status based on the
meter stype. For **electricity** meters, the representation is

| Bit   | Purpose                 | Description                                             |
| ----- | ----------------------- | ------------------------------------------------------- |
| Bit 7 | Reserved                | For future use                                          |
| Bit 6 | Service Disconnect Open | Set when the service has been disconnected              |
| Bit 5 | Leak Detect             | Bit is set when a leak is detected                      |
| Bit 4 | Power Quality           | Power quality event detected such as a low/high voltage |
| Bit 3 | Power Failure           | Power outage detected                                   |
| Bit 2 | Tamper Detect           | Bit is set when a tamper event is detected              |
| Bit 1 | Low Battery             | Battery needs maintenance                               |
| Bit 0 | Check Meter             | Errors such as measurement, memory or self-check error  |

For **gas** meters, the mstatus field can be understood as

| Bit   | Purpose            | Description                                                    |
|------ | ------------------ | -------------------------------------------------------------- |
| Bit 7 | Reverse Flow       | Flow detected in opposite direction, from consumer to supplier |
| Bit 6 | Service Disconnect | Set when the service has been disconnected                     |
| Bit 5 | Leak Detect        | Bit is set when a leak is detected                             |
| Bit 4 | Low Pressure       | Bit is set when a low pressure is detected                     |
| Bit 3 | Not Defined        |                                                                |
| Bit 2 | Tamper Detect      | Bit is set when a tamper event is detected                     |
| Bit 1 | Low Battery        | Battery needs maintenance                                      |
| Bit 0 | Check Meter        | Errors such as measurement, memory or self-check error         |

**aconsumindicate** has the following meanings

| Value         | Description         |
| ------------- | ------------------- |
| 0x00          | Low energy usage    |
| 0x01          | Medium energy usage |
| 0x02          | High energy usage   |

### Meter Info

The meter info API `get_meter_info` can be used to fetch basic meter information
as implied by the name. The following example demonstrates how it is used

```
import os
import requests

han_host = os.getenv('HAN_API_HOST')
meter_type = "elec"

# Call the get_meter_info API
response = requests.post(han_host + "/get_meter_info", json={"meter_type": meter_type})
if response.ok:
    meter_info = response.json()["meter_info"]
    print("Meter info for {} meter: {}".format(meter_type, meter_info))
else:
    print("Error calling get_meter_info API: {}".format(response.json()["Status"]))
```

this outputs data similar to

```
Meter info for elec meter: {
        "time": 1713538882,
        "minfo":        {
                "type": "elec",
                "eui":  "xxxxxxxxx",
                "serial":       "xxxxxx",
                "siteid":       "xxxxxx",
                "RSSI": -63
        }
}
```

where

| Variable | Description                                          |
| -------- | ---------------------------------------------------- |
| type     | Indicates meter type, gas/elec.                      |
| eui      | IEEE 64-bit address of the meter in ASCII Hex format |
| serial   | String representing the meter's serial number        |
| siteid   | String representing the meter’s site ID              |
| RSSI     | Will be explained later                              |

### Meter Consumption

The meter consumption API `get_meter_consumption` can be used to gather consumption
data. The following example demonstrates how it is used

```
import os
import requests

han_host = os.getenv('HAN_API_HOST')
meter_type = "elec"

# Call the get_meter_consumption API
response = requests.post(han_host + "/get_meter_consumption", json={"meter_type": meter_type})
if response.ok:
    meter_consumption = response.json()["meter_consump"]
    print("Meter consumption for {} meter: {}".format(meter_type, meter_consumption))
else:
    print("Error calling get_meter_consumption API: {}".format(response.json()["Status"]))
```

this outputs data similar to

```
Meter consumption for elec meter: {
        "time": 1713539551,
        "consum":       {
                "type": "elec",
                "consumption":  30117796,
                "instdmand":    284,
                "lastupdated":  766854750,
                "unit": 0,
                "raw":  {
                        "rawcons":      "00000000ABD0",
                        "rawinstdmand": "000000",
                        "multipler":    "000001",
                        "divisor":      "0003E8",
                        "consformat":   "33",
                        "demformat":    "00"
                }
        }
}
```

where

| Variable      | Description                                                                      |
| ------------- | -------------------------------------------------------------------------------  |
| type          | Indicates meter type, gas/elec.                                                  |
| consumption   | Current total consumption in Wh with multiplier, divisor and unit consideration  |
| instdemand    | Current demand in W with multiplier, divisor and unit consideration              |
| lastupadated  | UNIX timestamp indicating when the IHD received the data                         |
| unit          | Represents the unit of measure used for the raw data                             |
| consformat    | Recommendation on how to display the consumption number                          |
| demformat     | Represents a recommendation on how to display the demand number                  |
| rawcons       | Raw total consumption before multiplier, divisor and unit are applied            |
| multiplier    | 24-bit ASCII Hex number, used to calculate the consumption and instdemand values |
| divisor       | 24-bit ASCII Hex number, used to calculate the consumption and instdemand values |
| rawinstdemand | Raw instant demand before the multiplier, divisor and unit are applied           |

For further information and detailed clarification of these variables kindly
consult ZigBee Smart Energy Standard revision 1.1b.