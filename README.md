# vscp-python-digitemp
Digitemp to VSCP tcp/ip link script

If you have a Linux machine with one or several 1-Wire sensors connected to [digitemp](https://github.com/bcl/digitemp). This script can be used to automatically (preferably with the help of cron) report the temperature readings to a VSCP daemon with a tcp/ip link interface activated. 

By default VSCP GUID are setup using the 1-Wire id's for each sensors. This ensures globally unique id's.

The scripts works with both python version 2 and version 3.

## Setup digitemp

Install digitemp with

```
sudo apt install digitemp
```

### Serial adapter

Locate the serial port where the serial adapter is connected to.

Now run the following command in the same folder as you intend to execute the script.

```bash
digitemp -s/dev/ttyS2 -i
```

Change _/dev/ttyS2_ to the serial port you use.

Typical output

```bash
DigiTemp v3.7.2 Copyright 1996-2018 by Brian C. Lane
GNU General Public License v2.0 - http://www.digitemp.com
Found DS2490 device #1 at 001/004
Turning off all DS2409 Couplers
...
Searching the 1-Wire LAN
10A8AF9201080061 : DS1820/DS18S20/DS1920 Temperature Sensor
103D9D920108003C : DS1820/DS18S20/DS1920 Temperature Sensor
ROM #0 : 10A8AF9201080061
ROM #1 : 103D9D920108003C
Wrote .digitemprc
```

This will discover the 1-wire sensors connected to the adapter and create a file _.digitemprc_ in the same folder

### DS2490 USB adapter

You need to blacklist this adapter to make it work. Create a file _/etc/modprobe.d/ds2490-blacklist.conf_ and put

```bash
# ds2490 driver interferes with digitemp_DS2490 operation
blacklist ds2490
```

into it and restart the machine.

Now run the following command in the same folder as you intend to execute the script.

```bash
digitemp_DS2490 -i
```

Typical output

```bash
DigiTemp v3.7.2 Copyright 1996-2018 by Brian C. Lane
GNU General Public License v2.0 - http://www.digitemp.com
Found DS2490 device #1 at 001/004
Turning off all DS2409 Couplers
...
Searching the 1-Wire LAN
10A8AF9201080061 : DS1820/DS18S20/DS1920 Temperature Sensor
103D9D920108003C : DS1820/DS18S20/DS1920 Temperature Sensor
ROM #0 : 10A8AF9201080061
ROM #1 : 103D9D920108003C
Wrote .digitemprc
```

This will discover the 1-wire sensors connected to the adapter and create a file _.digitemprc_ in the same folder

## Change log format

Open the file _.digitemprc_ with your favorite editor and change the line starting with **LOG_FORMAT** to

```bash
LOG_FORMAT "%R %.2C"
```
**%R** is the sensor id and **%.2C** is the temperature in centigrade Celsius printed with two decimals.

## Sending data to the VSCP daemons tcp/ip link interface

If you run the command

```bash
digitemp_DS2490 -a -q
```

or 

```bash
digitemp_DS9097 -s/dev/ttyS2 -a -q
```

or 

```bash
digitemp_DS9097U -s/dev/ttyS2 -a -q
```

you will get something like

```bash
10A8AF9201080061 15.38
103D9D920108003C 23.38
```

This is exactly what _sendvalue.py_ want as input so

```bash
digitemp_DS2490 -a -q | ./sendvalues.py 192.168.1.7 admin secret
```

will send the temperatures to the server _192.168.1.7_ with credentials user="admin" and password="secret".

# Using cron

To get temperature automatically sent to the server on even intervals cron can be used.  Create a file _dodigitemp_ in _/etc/crontab.d_ with the following content

```bash
* * * * * root cd /root;digitemp_DS2490 -a -q | ./sendvalues.py 192.168.1.7 admin secret
```

This will send temperatures to the VSCP daemon every minute. Change cron interval to suite your case.

**Note**: To secure your credentials you can enter them in the sendvalue.py script instead. With the above username and password will be exposed on a multiuser machine.
