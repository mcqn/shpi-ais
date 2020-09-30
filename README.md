# shpi-ais

Raspberry Pi RTL-SDR for Ship AIS tracking.

For more background, [Ranous' Linux Quickstart guide](https://ranous.files.wordpress.com/2018/02/rtl-sdr4linux_quickstartv2-18.pdf) is useful.

## Setup

1. Get a Raspberry Pi
1. Buy an RTL-SDR dongle, ship antenna and appropriate cable to connect the two (this has been tested with [this SDR dongle](https://www.rtl-sdr.com/buy-rtl-sdr-dvb-t-dongles/) with [this antenna](https://www.nevadaradio.co.uk/product/sirio-sb-1-s/))
1. Connect the antenna to the SDR dongle
1. Plug the dongle into one of the USB ports on the Raspberry Pi
1. Install the software as below

## Installation

1. Install [Ansible](https://www.ansible.com/get-started) on your computer
1. Install the latest [Raspbian lite image](https://www.raspberrypi.org/downloads/raspbian/) onto a micro-SD card
1. Create a file called `ssh` on the `/boot` partition of the SD card.  The contents don't matter, it's just to enable the SSH server.  E.g. on Ubuntu `touch /media/myusername/boot/ssh`
1. Boot the Raspberry Pi with the micro-SD card, while plugged into a network via Ethernet
1. Find out the IP address of the Raspberry Pi
 * Use nmap (eg: `nmap -p 22 192.168.0.* --open`), router or monitor to find IP address of Pi once booted.
1. Copy your SSH credentials onto the Pi
  ```ssh-copy-id pi@<ip-address-of-the-pi>```
1. Edit the ```hosts``` file so ansible knows which computer to configure.  Change the IP address in it to match the one you just found out.
1. Check you can run commands on the Pi using Ansible
   ```ansible controlled-pi -i hosts -a "hostname" -u pi```
1. Run the playbook to install the relevant software and configure the Pi
   ```ansible-playbook headless-shpi-ais.yml -i hosts```

## Usage

For basic output of any received AIS packets to stdout:
```
cd ~/rtl-ais
./rtl_ais -n
```

### GnuAIS

`rtl_ais` will print out the raw AIS packets, but doesn't decode them.  GnuAIS will decode and output the information contained in them.  We can use a named pipe to tie the two together:

```
mkfifo aispipe
./rtl_ais -A aispipe
```
In another terminal, hook `gnuais` up to the other end of the pipe
```
gnuais -l aispipe
```
If you're running on a full version of Raspbian with a GUI, you can install `gnuaisgui` (run `sudo apt install gnuaisgui`) and then run that, after getting `gnuais` running, and it will show the position of the ships on a map.

[!Screenshot of gnuaisgui showing a collection on ships on the River Mersey in Liverpool](gnuguiais-screenshot.png)
