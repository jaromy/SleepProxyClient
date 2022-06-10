# A SleepProxyClient implementation

## About

Wake on Demand (http://support.apple.com/kb/HT3774) is great.
It enables an unused device to go to sleep while keeping the announced MDNS (Zeroconf) services alive.
Just access one of the services and the device will be woken up again.

These scripts enables your Non-Apple server (or NAS) to save energy by going to sleep if it's currently not in use.
But it will be instantly woken up again by the SleepProxyServer using Wake on Lan (WOL) if one of it's services is requested. See http://en.wikipedia.org/wiki/Bonjour_Sleep_Proxy for more details.

### Requirements
To get this running, a SleepProxyServer on your local network is required. If present, it will announce itself via MDNS as <code>_sleep-proxy._udp</code>. 
Such a server is included in many Apple devices like its network products "Time Capsule" and "AirPort Express". But an Apple TV or any Mac running 10.6 or above can be turned into a sleep proxy server too.

### Status
SleepProxyClient is pretty stable and ready for everyday use.
Please report issues to make it even more stable.


## Setup / Install

### Requirements

 - python 3
 - dnspython (http://www.dnspython.org)
 - and other usefull python modules
 - avahi-browse (to discover the sleep proxy and local services)
 - pm-utils or similar power management tools

 In addition it has to be possible to wake the host via Wake on LAN from sleep.
 
### Install

SleepProxyClient works out of the box on Debian/Ubuntu.
It should be quite easy to install SPC on other Linux distributions and UNIX/BSD systems too.

#### Linux
I've only installed this on Ubuntu and Xubuntu. Install manually, or via install script `debian/install`. Either way, it's pretty easy:

  * Clone the repository

```bash
git clone https://github.com/jaromy/SleepProxyClient.git .
```

Manual:
  * Copy files to appropriate directories:

```bash
mkdir -p /usr/share/sleepproxyclient/scripts
sudo cp {sleepproxyclient.sh,sleepproxyclient.py,checkSleep.sh} /usr/share/sleepproxyclient/scripts
sudo cp sleepproxyclient-systemd /lib/systemd/system-sleep/
sudo cp debian/sleepproxyclient.default /etc/default/sleepproxyclient
```

You'll also need to set appropriate permissions


#### Other operating systems

SleepProxyClient was tested on Linux but should work on other operating systems too.  
The only Linux specific dependency `pm-utils` should be easily replaceable by other OS-specific power management tools like `apmd` on BSD.
Just ensure the suspend-script (`sleepproxyclient.sh`) is called before suspending the system.


### Configuration

#### Services

All locally announced services will be discovered via avahi-browse. There is no manual configuration needed anymore.
However, a filter has been added in `sleepproxyclient.py` to allow the filtering of specific services. Only these designated services will be registered with the Sleep Proxy Server. This is to circumvent the issue of the registration failing due to the maximum message length exceeding 512 bytes.

#### Setup auto-sleep

Create your own or use the included <code>checkSleep.sh</code> script.
The checkSleep script needs to be called periodically to be able to suspend the host by it's own.
This can be done by creating a cronjob via crontab:
<pre>*/8 * * * * /bin/bash /usr/share/sleepproxyclient/scripts/checkSleep.sh</pre>

This job causes the checkSleep.sh to be called every 8 minutes. Since two successfully calls are required to suspend the host it will take at least 16 minutes until the suspend is done.

#### Tuning some parameters

Some more parameters can be adjusted to fit your needs:

- List of network interfaces    
	By default all interfaces are used. This can be changed by enabling this option.

- TTL (Time to live)   
	The TTL controls the life time of the mDNS announcement. After this period the sleep client will be woken up be the sleep proxy server again. The default value is 2 hours.

These settings can be configured via <code>/etc/default/sleepproxyclient</code>
	
### What's inside

- 00_sleepproxyclient    
	This hook will be installed to <code>/etc/pm/sleep.d/</code> and called by pm-utils before going to sleep. This script will call sleepproxyclient.sh and is actually calling the sleepproxyclient scripts.

- sleepproxyclient-systemd
	This hook is for systemd compatible systems (e.g. recent Ubuntu). It should be installed into <code>/lib/systemd/system-sleep</code> and given appropriate executable privileges. This script will call sleepproxyclient.sh to handle the work. It will also restart the avahi daemon service upon wake-up.

- checkSleep.sh   
 Is an example script to show how to actually suspend the host. It does some checks to determine if the host is currently in use or not. It will suspend the host after two successfully calls by <code>pm-suspend</code>. This script is designed to be periodically called by a cronjob.
	To be able to do some more advanced checks take a look at other projects like https://github.com/OMV-Plugins/autoshutdown/. Just configure them to call <code>pm-suspend</code> instead of <code>shutdown</code> to activate your SleepProxyClient.

