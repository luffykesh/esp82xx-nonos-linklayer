
# lwIP-2 from original source

This repo offers a link layer for esp82xx-nonos-sdk-2.
The original goal is to try and use a recent lwIP version for stability reasons.
Currently lwIP-v2 is implemented, other IP stacks could be tried.

# Note

* ipv6 not tried yet
* tcp is more stable
* needs testing

# Tested to work so far

* NTPClient
* WiFiAccessPoint
* OTA
* seems to solve some TCP issues

# rebuild

makefiles are working with linux/osx, and maybe with windows (using 'make' included in msys from mingw...)

```
cd <path-to-your>/esp8266/tools/sdk/lwip2/builder
```

get lwIP sources
```
./lwip2-update-stable
```

optionnally tune lwIP configuration in tools/sdk/lwip2/builder/glue-lwip2/lwipopts.h

build & install
```
make install
```

this will overwrite tools/sdk/{lib/liblwip2.a,lwip2/include/}

# about MSS

Remember the MSS footprint: 4*MSS bytes in RAM per tcp connection.
The lowest recommanded value is 536 which is the default here.

# How it works

Espressif binary libraries rely on their lwip implementation. The idea, as
described in this [comment](https://github.com/kadamski/esp-lwip/issues/8)
is to wrap espressif calls, and rewrite them for a new tcp implementation.

Example with lwip1.4's ethernet_input() called by espressif binary blobs
finally reaching lwip2's:

```
-- LWIP2-----------------------------------
#define ethernet_input ethernet_input_LWIP2
- lwip2's ethernet_input_LWIP2_is called
                            (/ \)
                             | |
-- glue (new side)-----------^-v-----------
                             | |
glue_ethernet_input          | |
- maps structures glue->new  |
- calls ethernet_input_LWIP2(^ v)
- maps structures new->glue    |
                             | |
-- glue (old side)-----------^-v-----------
                             | |
ethernet_input():            | |
- maps structures old->glue  | 
- calls glue_ethernet_input (^ v)
- maps structures glue->old    |
                             | |
- espressif blobs -----------^-v-----------
XXXXXXXXXXXXXXXXXXXXXXXXXXXX | | XXXXXXXXXX
wifi calls    ethernet_input(/ \) XXXXXXXXX
XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
-------------------------------------------
```
