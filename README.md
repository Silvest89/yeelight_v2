# yeelight_v2

This integration and its accompanying submodules (libraries) can be used as an alternative yeelight integration (custom component) for Home Assistant.
It currently supports SSDP as a fallback for the get_prop method.
The miio protocol has also been implemented. If a miio_token is provided then the miio protocol will be used to communicate with the bulb (no LAN Control needed)

# Getting started

```
Create a custom_components folder in the root of your HA folder (where configuration.yaml resides)
cd into custom_components
git clone --recursive https://github.com/Silvest89/yeelight_v2.git
Restart HA
```

I'd recommend the config flow to set it all up since this way new config properties can be changed on the fly. 
Else you'd have to remove the bulbs/integration and add it again each time new config properties are added

Should you prefer the old way (editing configuration.yaml)

Add:
```
yeelight_v2:
  devices:
    192.168.1.10:
      name: "Bulb"
      ssdp_fallback: True # Recommended, the default value is False
      miio_token: "" # if provided miio protocol is used (optional)
```
# Logs
```
logger:
  default: critical
  logs:
    custom_components.yeelight_v2: debug
    custom_components.yeelight_v2.python_yeelight.yeelight.main: debug
```

# Getting tokens
See below a sample Dockerfile containing everything needed to extract tokens. Create a folder and put the contents in the Dockerfile

```
FROM python:3.10.0a4-buster

COPY . /data
RUN pip3 install python-miio
```

## Android

```
* Connect your android device to your pc
* adb backup -noapk com.yeelight.cherry -f backup.ab
* Put the backup file in same folder as the Dockerfile
* docker build -t silvest/miio-extract .
* docker run --rm silvest/miio-extract miio-extract-tokens /data/backup.ab
```

## iOS
For iOS backups I'd refer to https://python-miio.readthedocs.io/en/latest/discovery.html#creating-a-backup
this page talks about the mihome app but it should work the same for yeelight.
The yeelight appdomain = com.yeelight.cherry

### Sample response from miio-extract-tokens
```
miio-extract-tokens /tmp/yeelight.ab --password a

Unable to find miio database file apps/com.xiaomi.smarthome/db/miio2.db: "filename 'apps/com.xiaomi.smarthome/db/miio2.db' not found"
INFO:miio.extract_tokens:Trying to read apps/com.yeelight.cherry/sp/miot.xml
INFO:miio.extract_tokens:Reading tokens from Yeelight Android DB
Yeelight Color Bulb
        Model: yeelink.light.color1
        IP address: 192.168.xx.xx
        Token: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
        MAC: F0:B4:29:xx:xx:xx
Mi Bedside Lamp
        Model: yeelink.light.bslamp1
        IP address: 192.168.xx.xx
        Token: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
        MAC: 7C:49:EB:xx:xx:xx
```

# VLAN/Cross Subnets
## SSDP
For SSDP to work across vlans/subnets you'd have to have an MDNS repeater (Opnsense), Avahi (Pfsense) or any equivalent if you don't use those.

## MIIO
As for the miio procol you'd need to setup MASQUERADE so rewrite the source adress to be the interface address.
My guess is still that it is a "security" feature where the device will not respond to messages not coming from the same subnet.
If you've multiple VLAN's/subnets and you're in control over the router in between, then I'd setup masquerading for the outgoing routing interface of the VLAN/subnet where the Xiaomi devices reside. This basically means changing the source address in the UDP packet headers to the IP address of routing interface. If you want to know more about this, just inform yourself about packet masquerading and/or SNAT.
