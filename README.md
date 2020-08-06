# RetroPie 4.6 and PS2SMB Guide

This guide is to have a working RetroPie setup whilst also using PS2SMB network 
share to play PlayStation 2 games via ethernet. This guide also requires your 
PS2 to have a memory card flashed with FreeMCBoot and OPL.

# Step 1 - Preparing the SD Card

Firstly, download the latest image of [RetroPie](https://retropie.org.uk/download/) and extract the iso file.

Format your micro SD card to FAT32. If it's larger than 32GB, you'll have to use a tool 
like [guiformat](http://www.ridgecrop.demon.co.uk/index.htm?guiformat.htm) to do this.

After formatting, use [Balena Etcher](https://www.balena.io/etcher/) and select the RetroPie 
iso you downloaded earlier with the micro SD card. This will take a couple minutes. It's recommended 
to have an 8GB or larger micro SD card.

After this is done, you've set up the SD card!

# Step 2 - Configuring RetroPie

Slot the micro SD card into the Raspberry Pi 4 and connect the display, power and a controller.
For HDMI audio, only one of the ports supports this. Turn the power on and you should see 4 raspberries 
appear in the top corner.

After the initial boot, the device will restart by itself. It will be ready when it prompts for controller 
mapping. Perform the manual controller mapping and you're good to go.

On a seperate usb, create a folder named "retropie" in the root directory. Plug this into the 
Raspberry Pi and when it has finished flashing, eject the USB. the "retropie" folder on the usb should now 
contain folders for roms, which you can add to their respective folder. A bios folder is also present if 
you're using an emulator that requires a bios, such as PS1 or Dreamcast.

For some small optimisations, if your screen has black borders around the edges, you can modify this by 
exiting EmulationStation (Start Button -> Quit -> Quit EmulationStation -> Yes), which will leave you in 
the terminal. Type:
```sudo nano /boot/config.txt"```
And find the line "disable_overscan=0". The number may be different, but experiment with adjusting this 
value (changing this to 1 fixed any black borders for my setup) and rebooting by typing:
```reboot```

# Step 3 - Setting up a USB for PS2 Games

For this, you'll need a fairly large USB formatted to NTFS format. As PS2 games tend to be around the 4GB 
mark, a larger USB helps store more games. Technically you could do this with a FAT32 usb, but games 
larger than 4GB will need to be split into parts. This USB needs folders in the root directory that are 
specified [here](https://bitbucket.org/ShaolinAssassin/open-ps2-loader-0.9.3-documentation-project/wiki/tree-structure). 
You can do this by plugging the USB into the PS2 and launching OPL to create the directory paths or manually. 

After this, plug the USB into your Raspberry Pi and enter the terminal by quitting EmulationStation. 

# Step 4 - Setting up the Raspberry Pi to share PS2 Games over ethernet

Once in the terminal, type: 
```sudo nano /etc/dhcpcd.conf```
and find the eth0 setting. These should be commented out by default. Uncomment the following lines to look 
something like this:
```
interface eth0
static ip_address=192.168.20.10/24
static routers=192.168.20.1
static domain_name_servers=192.168.20.1
```
To exit, press Ctrl+X -> Y -> Enter

Do note that the specific numbers will be different for your case. Ensure that ip_address is a unique number. 

After this, reboot the system with: 
```reboot```

After this, we need to enter the terminal again. Once in, we need to find our USB. Type:
```sudo fdisk -l```
Multiple entries will appear. Generally the last one is the USB. Looking at the Device boot column, you'll see 
the path to the device, in most cases being **/dev/sda1**.

Next we need to create a folder to mount the drive which can done with a command like:
```sudo mkdir /media/PS2SMB```

We also need to modify the stab config file which can be done with the command:
```sudo nano /etc/fstab```
Then after the last entry, add a newline and add the following line: 
```/dev/sda1 /media/PS2SMB ntfs default,nofail 0 0```

To exit, press Ctrl+X -> Y -> Enter.

After this, reboot the system with: 
```reboot```

We need to do one last thing, which is to setup the Samba server to make our Raspberry Pi act as a server.
Samba is already installed in RetroPie, and we need to simply add some lines to the config. To do this, in the terminal type:
```sudo nano /etc/samba/smb.conf```
Under the ```[global]``` header, add the lines:
```
lanman auth = yes
keepalive = 0
```
and at the very bottom add the following chunk:
```
[PS2SMB]
comment = PS2 folder
path = /media/PS2SMB
browseable = yes
writeable = yes
only guest = no
guest ok = yes
create mask = 0777
directory mask = 0777
public = yes
```

To exit, press Ctrl+X -> Y -> Enter. After editing, you can check whether your syntax is correct using command:
```testparm```

After this, restart the samba server using command:
```sudo service smbd restart```

# Step 5 - Setting up OPL on the PS2

Connect your Raspberry Pi 4 and PS2 together using an ethernet cable and turn on the PS2. With the Free McBoot 
memory card, open OPL and open up Network config. From here change the settings to the following:
```
PS2
IP Address Type: Static
IP Address: 192.168.20.xx
Mask: 255.255.255.0
Gateway: 192.168.20.10
DNS: 192.168.20.10

SMB Server
Address Type: IP
IP Address: 192.168.20.10
Share: PS2SMB
User: guest
```



# NOTES

Is USB auto mount neccessary?
Samba 



