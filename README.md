# Promiscuous Bluetooth audio sinc

A2DP agent for promiscuous/permissive audio sinc for Linux. Once installed, a Bluetooth client, such as a smart phone, should be able to discover, pair, and subsequently play audio without any manual interaction. This is perfect for those with headless media boxes wanting to expand their connective options and saves explaining things to the kids 8) 

This project assumes the use of PulseAudio and should be tested with PortAudio if required.

This project is heavily based on the Gist and comments at https://gist.github.com/mill1000/74c7473ee3b4a5b13f6325e9994ff84c#gistcomment-4032842

## Authenticating
This agent essentially allows for non-interactive pairing with some caveats.

When a pairing client requests a _pairing code_, simply accept the code.

When a pairing client requests a pin code set as `0000`.

## Installation

**NOTE:** This is very much a WIP, **please** comment on any additional steps required in your distro/version.

This documentation is based on my experience using Ubuntu 21.04.

### Install required packages

Ensure that the required Bluetooth, Bluetooth audio, and Python 3 dependencies are installed.

```bash
sudo apt install pulseaudio-module-bluetooth bluez
```

### Install the A2DP Bluetooth agent

A Bluetooth agent is a piece of software that handles 
pairing and authorization of Bluetooth devices. The following agent 
allows a client to automatically pair and accept A2DP 
connections from Bluetooth devices.
All other Bluetooth services are rejected.

Copy the included file **a2dp-agent** to `/usr/local/bin` and make the file executable with

```
sudo cp -a a2dp-agent /usr/local/bin/
sudo chmod +x /usr/local/bin/a2dp-agent
```

#### Testing the agent

Before continuing, verify that the agent is functional. 
Bluetooth audio should be discoverable, pairable and recognised as an 
audio device.

1. Manually run the agent by executing

```
sudo /usr/local/bin/a2dp-agent
```

2. Attempt to pair and connect with using your phone or computer. You will be asked to verify a pairing key, just accept it and move on.
3. Check on the client which profiles are available. `SBC` is likely your best choice quality wise. If you do not see any services then something has gone wrong 8(
4. The agent should output the accepted and rejected Bluetooth UUIDs

```
Setting device to 'discoverable' and 'pairable'...
Agent registered
RequestConfirmation (/org/bluez/hci0/dev_04_B1_67_56_62_C3, 263837)
Auto confirming...
AuthorizeService (/org/bluez/hci0/dev_04_B1_67_56_62_C3, 0000110d-0000-1000-8000-00805f9b34fb)
Authorized A2DP Service
```

Play some audio on the client device, you should hear or see something happening on the server side 8)

## Install the A2DP Bluetooth Agent as a service

To make the A2DP Bluetooth Agent run on boot copy the included file **bt-agent-a2dp.service** to `/etc/systemd/system`.
Now run the following command to enable the A2DP Agent service

```
sudo cp -a bt-agent-a2dp.service /etc/systemd/system/
sudo systemctl enable bt-agent-a2dp.service
sudo systemctl start bt-agent-a2dp.service
systemctl status bt-agent-a2dp.service
```

The `status` should show green:

```
● bt-agent-a2dp.service - A2DP Bluetooth Agent
     Loaded: loaded (/etc/systemd/system/bt-agent-a2dp.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2022-01-19 10:26:30 NZDT; 1s ago
   Main PID: 17527 (python3)
      Tasks: 2 (limit: 18760)
     Memory: 8.9M
        CPU: 55ms
     CGroup: /system.slice/bt-agent-a2dp.service
             └─17527 /usr/bin/python3 -u /usr/local/bin/a2dp-agent
```

You can follow the logs of the agent with:

```bash
journalctl -fu bt-agent-a2dp.service
```

## Configuring the Bluetooth adapter
By default `hci0` is set to discoverable and pariable.
In the case where there are multiple Bluetooth adapters, one or more adapters can be set to discoverable and pairable in the configuration file `/etc/default/a2dp-agent` as a comma separated list of devices as:

```bash
BLUETOOTH_ADAPTER=hci1
```
or
```bash
BLUETOOTH_ADAPTER=hci0,hci1
```

## Higher quality audio

SBC has a bad reputation compared with other codes, but it turns out that good old SBC has a _dual channel_ mode which can be enabled in the _Developer options_ in recent LineageOS based Android phones.  See [Bluetooth SBC Dual Channel HD audio mode – LineageOS – LineageOS Android Distribution](https://lineageos.org/engineering/Bluetooth-SBC-XQ/) for a breakdown.
