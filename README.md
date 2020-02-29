
# Raspberry Pi 4 NAS Server with OpenMediaVault 5 and NextCloud on MacOS Mojave

## Setup
- Raspberry Pi 4 with Raspbian
- External hard drive (I'm using the ADATA HD710 1TB with ExFAT and USB 3.0)
- Keyboard and mouse (to connect to RPI)
- Mini HDMI (to connect to monitor)

## 1. Enabling SSH on RPI

Skip this section if you already have SSH setup on your RPI.

### Enable SSH on RPI

On your RPI, go to Menu > Preferences > Raspberry Pi Configuration. Go to the Interfaces panel and make sure SSH is Enabled.

### Get your IP address

On your RPI, open a terminal window and enter the following command:
```
hostname -I
```
which will return your RPI IP address. For example,
```
192.168.0.41
```
copy this value down somewhere.

### Connect to RPI via SSH

Open a terminal window on your Mac computer and enter the following command:
```
ssh pi@192.168.0.41
```
The IP address is the example we used above. Replace the `192.168.0.41` with the IP address your own IP address you got in the previous step.

Terminal will ask you if you want to continue. Enter `yes` and press RETURN. At this point, you'll be asked to enter the RPI password.

You are now connected to your RPI via SSH. To prove this, enter the command `ls` and you will see a list of the root RPI files.

To make this easier the next time you want to connect to your PI, set up an alias on your Mac. Instructions for this can be found online. For example, the alias I set up in .bash_profile is `alias pi="ssh pi@192.168.0.41"`

## 2. Installing OMV5

DISCLAIMER: The following instructions came from Wagner's Tech Talk (http://wagnerstechtalk.com/rpi4omv/) and it's how I learned how to install OMV5 beta on my RPI. Everyone say thank you to Wagner!

In the Mac Terminal (which is connected to the RPI now via SSH) enter each of the following commands (each command is the entire code block, ranging from 1 to several lines) and press enter each time. This should go without saying, but make sure to wait until the previous command process is done before entering the next one.
```
sudo su
```

```
cat <<EOF >> /etc/apt/sources.list.d/openmediavault.list
deb https://packages.openmediavault.org/public usul main
# deb https://downloads.sourceforge.net/project/openmediavault/packages usul main
## Uncomment the following line to add software from the proposed repository.
# deb https://packages.openmediavault.org/public usul-proposed main
# deb https://downloads.sourceforge.net/project/openmediavault/packages usul-proposed main
## This software is not part of OpenMediaVault, but is offered by third-party
## developers as a service to OpenMediaVault users.
# deb https://packages.openmediavault.org/public usul partner
# deb https://downloads.sourceforge.net/project/openmediavault/packages usul partner
EOF
```

```
export LANG=C.UTF-8
```

```
export DEBIAN_FRONTEND=noninteractive
```

```
wget -O "/etc/apt/trusted.gpg.d/openmediavault-archive-keyring.asc" https://packages.openmediavault.org/public/archive.key
```

```
apt-key add "/etc/apt/trusted.gpg.d/openmediavault-archive-keyring.asc"
```

```
apt-get update
```

```
apt-get --yes --auto-remove --show-upgraded \
	--allow-downgrades --allow-change-held-packages \
	--no-install-recommends \
	--option Dpkg::Options::="--force-confdef" \
	--option DPkg::Options::="--force-confold" \
	install openmediavault-keyring openmediavault
```

```
omv-confdbadm populate
```

```
cat /etc/issue
```
--

OMV5 should now be ready to go on your RPI. You can access OMV5 on your Mac by opening up a web browser and entering in the following URL:
```
http://<hostname>
```
where `<hostname>` should be replaced by the hostname of your RPI. You can get the hostname of your RPI by entering the `hostname` command in your RPI terminal (or SSH connected one). 














