
# How to build a personal cloud storage server in 1 day or less.

#### The instructions included in this tutorial are for those working on MacOS. If are working on a Microsoft system or similar the general process is the same, you'll just have to convert the code instructions to the non-linux equivalent.

### Preface

This tutorial was created on March 1st, 2020. I will not be updating it much in the future, however, unless there are major changes made to OpenMediaVault or NextCloud, it should still be applicable for years to come.
<br />
<br />
Also, big shoutout to <a href="https://pcmac.biz/">PcMac</a> and <a href="http://wagnerstechtalk.com/rpi4omv/">Wagner's TechTalk</a>, whose videos and documentation taught me how to do this. If you learn better through YouTube videos, check out their YouTube pages <a href="https://www.youtube.com/channel/UCAflQPuKZnTBcLZIbPYCyaA/featured">here (PcMac)</a> and <a href="https://www.youtube.com/channel/UC1QvDo-qeCVKTlmDq1LjDuw">here (WTT)</a>.
<br />
<br />
If you're ready to build your own personal cloud-based storage solution, let's get started!

### What you're gonna need (or something similar)
- Mac computer (this is an Apple tutorial)
- Raspberry Pi 4 with Raspbian
- Micro SD card (ideally >4GB and FAST)
- External hard drive (this is your actual storage, so ideally USB 3.0, >500GB and formatted as ExFAT)
- Keyboard and mouse (to connect to RPI, optional)
- Mini HDMI (to connect to monitor)
- OpenMediaVault 5 (usul)
- NextCloud V18.0.1
- PHP V7.3.14

## 1. Enable SSH on RPI (Optional)

Setting up SSH is optional, however it will make your life easier because we will be able to take full advantage of the bandwidth and power of your Mac, as opposed to doing everything on the RPI. Your RPI will become headless, too.

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

## 2. Install and Setup OMV5

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

OMV5 should now be ready to go on your RPI. You can access OMV5 on your Mac by opening up a web browser and entering in the following URL:
```
http://<hostname>
```
where `<hostname>` should be replaced by the hostname of your RPI. You can get the hostname of your RPI by entering the `hostname` command in your RPI terminal (or SSH connected one). If this doesn't work (most modern browsers automatically prefix `www` and suffix `.com`) you can simply enter the IP address obtained above.

### Troubleshooting

If after installing OMV5 you can no longer log into your RPI via SSH (`permission denied (public key, password)`) it is because the `pi` user is not included in the `ssh` group on OMV5. To fix this, go to the OMV5 GUI, navigate to the Group tab, select the pi group from the table and click Edit. Add the pi user to the pi group, then select OK. Navigate to the User tab, select the newly created pi user and click Edit. Under the Groups tab, scroll down and make sure SSH is enabled (checked). Save and apply settings and you should be able to SSH to your RPI again.

## 3. Install NextCloud on top of OMV5

Navigate to OMV by going to `http://<hostname>` and enter your login information. Unless you have changed it, the username is `admin` and password is `openmediavault` by default.

Change the port to 90 (default is 80). You will get an error because OMV5 is only available on port 90 now and you need to specify this in the URL (<IP>:90). We do this because Apache web server defaults to port 80 (explained later on) and it's just easier this way.
	
Before continuing, make sure to `apt-update` and `apt-upgrade`.
	
### Install Web Server (Apache), MariaDB Server, and PHP modules

Enter these commands into your SSH-connected Terminal:
```
sudo apt-get install apache2 mariadb-server libapache2-mod-php
```
then
```
sudo apt-get install php7.3-gd php7.3-json php7.3-mysql php7.3-curl php7.3-mbstring 
```
then
```
sudo apt-get install php7.3-intl php-imagick php7.3-xml php7.3-zip
```

Finally, restart the web server
```
sudo service apache2 restart
```
Go to the OMV5 GUI port 80 (default) and the default Apache web page should show up. This is good.

### Download NextCloud

Time to add NC! First, navigate to the `html` directory with the following command:
```
cd /var/www/html
```
Then run the following Curl script to download NC 18:
```
curl https://download.nextcloud.com/server/releases/nextcloud-18.0.1.tar.bz2 | sudo tar -jxv
```
Now we create a folder where NC can save information.
```
sudo mkdir -p /var/www/html/nextcloud/data
```
Next, we configure control and permissions of our newly created folder so we can access NC on our webserver.
```
sudo chown -R www-data:www-data /var/www/html/nextcloud/
```
```
sudo chmod 750 /var/www/html/nextcloud/data
```
Now that we're done, restart the Apache web server and reboot your RPI.
```
sudo service apache2 restart
```
```
sudo reboot
```
Give your RPI a chance to reboot, then login via SSH again. Run `apt-update` and `apt-upgrade` to make sure everything is updated.

Now, the exciting part. Go to your `<ip>` and you should see the Apache default page runnning. Suffix the URL with `/nextcloud/` (Ex. `192.168.1.41/nextcloud`) and you should see the NC login page. 

## 4. Configure NextCloud

In order to configure NC, we need to configure the MySQL database. Back to the SSH Terminal we go. Change all of the code in `[brackets]` to your own text.

```
sudo mysql
```

```
CREATE DATABASE [db name] CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
```

```
GRANT ALL ON [db name].* TO '[db username]'@'localhost' IDENTIFIED BY '[strong password]';
```

```
FLUSH PRIVILEGES;
```

```
EXIT;
```

## 5. Allow NextCloud Access from External Networks

### Port Forwarding

We need to forward <b>Port 80</b> and <b>Port 443 (for Dynamic DNS, explained later on)</b> to the RPI IP address.

To set up port forwarding, you'll need to access your router admin. To access your router, type its IP address into your browser. If you don't know the IP address, use the following command to find it:

```
netstat -nr | grep default
```
It will be listed beside the `default` keyword and will probably look something like `192.168.1.1`. Because you probably have a different router than me, I am unable to help you with this step. Google the instructions on how to set up port forwarding for your particular router model.

### Dynamic DNS

To set up dynamic DNS, we will use <a href="https://www.duckdns.org/">Duck DNS</a>. It's free and straightforward to use. At this point, you should make sure that Port Forwarding is set up for Port 443, or this isn't going to work.
Log in to Duck DNS using one of the provided methods, then add a domain of your choice, for example `mycloud.duckdns.com`.

#### NOTE: You might not be able to access your `.duckdns.org` domain on your wifi network. For testing, use your phone one cellular data. You should see the apache default page if everything is working correctly.

### Configuring trusted domains

In order to access your NC, we need to configure the `.duckdns.org` domain in the `config.php` file. Time to go back to the terminal. Through SSH, navigate to `/var/www/html/nextcloud/config` using the following command:

```
cd /var/www/html/nextcloud/config
```

Next, we need to edit the `config.php` file. `config.php` has heavy permissions so you will need to `sudo` in order to read & write. Use the following command:

```
sudo nano config.php
```

The contents of the file should look something like this:
```
<?php
$CONFIG = array (
  'instanceid' => 'xxx',
  'passwordsalt' => 'xxx',
  'secret' => 'xxx',
  'trusted_domains' =>
  array (
    0 => '192.168.x.x',
  ),
  'datadirectory' => 'xxx',
  'dbtype' => 'xxx',
  'version' => 'xxx',
  'overwrite.cli.url' => 'xxx',
  'dbname' => 'xxx',
  'dbhost' => 'xxx',
  'dbport' => '',
  'dbtableprefix' => 'oc_',
  'mysql.utf8mb4' => true,
  'dbuser' => 'xxx',
  'dbpassword' => 'xxx',
  'installed' => true,
);
```
We need to edit the `trusted_domains` property and add our `.duckdns.org` domain name. After doing so, the file should look like this:
```
<?php
$CONFIG = array (
  'instanceid' => 'xxx',
  'passwordsalt' => 'xxx',
  'secret' => 'xxx',
  'trusted_domains' =>
  array (
    0 => '192.168.x.x',
    1 => 'mycloud.duckdns.org',
  ),
  'datadirectory' => 'xxx',
  'dbtype' => 'xxx',
  'version' => 'xxx',
  'overwrite.cli.url' => 'xxx',
  'dbname' => 'xxx',
  'dbhost' => 'xxx',
  'dbport' => '',
  'dbtableprefix' => 'oc_',
  'mysql.utf8mb4' => true,
  'dbuser' => 'xxx',
  'dbpassword' => 'xxx',
  'installed' => true,
);
```
Make sure to save the file. Once that is done, you'll be able to access your NC via any network using the `.duckdns.org` domain name. Give it a try on your phone to confirm that it works (use cellular data again).


## 6. Add External Hard Drive and Connect it to NextCloud

Ah, the final step (thank goodness).

On the OMV dashboard, navigate to File Systems. From here, find your connected HD and click the Mount button.

Now, on the NC dashboard, click the icon at the top-right of your screen and select Apps from the dropdown menu. Scroll down to the External Storage Support row and enable it.

### Configure Permissions

Back in our SSH Terminal, we need to set permissions on our newly mounted HD. First, we need the mount point path. We can find this by going into the `/srv/` directory and typing `ls` to view the folder contents. You should see something like `dev-disk-by-label-XXX`. The mount point path is therefore `/srv/dev-disk-by-label-XXX`.

Now that we have this, enter the following commands to configure a couple permissions:
```
sudo chown -R www-data:www-data /srv/dev-disk-by-label-XXX
```
```
sudo chmod -R 0750 /srv/dev-disk-by-label-XXX
```
Back in the NC dashboard, click the top-right icon again and then go to Settings. Add a folder name of your choice, set the External Storage to Local and paste in the `/srv/dev-disk-by-label-XXX` location under Configuration. Save it, and now if you navigate back to the External Storages tab you should see your newly added HD and its data.

### Connect it to OMV

In the OMV dashboard, under Shared Folders, add a shared folder with the mounter HD as the Device. Then, under SMB/CIFS, make sure it is enabled (under General Settings) and click on the Shares tab. Add a share using the folder we just created and configure it as you like.

### Time to test

We're done! Let's make sure everything is working and we can move files from our Mac to our External Hard Drive.

Open up a Finder window on your Mac and press `Command+K`. In the text field, enter the following:
```
smb://<IP of your RPI>
```
Connect to it and enter your login information. Once it's all set up, try to drag & drop some files into the folder. Wait for them to copy and Wall-Ah! Your files are safe & sound on your HD and can be accessed through NextCloud.

## 7. Celebrate

Congratualtions! You just created your own personal cloud storage solution and you can now put files directly onto your external hard drive from anywhere in the world. It's not the most performant solution in the world but hey, you're in complete control of your data now. Give yourself a pat on the back, you deserve it.




