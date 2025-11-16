# Debian-setup
creating my first headless server

1. 
Install Debian iso from https://www.debian.org/

Downlaod and format the installation media. Dependant on UEFI or BIOS system you need to chose GPT or MBR partition scheme.
  
We do this with a tool called RUFUS on Windows. https://rufus.ie/en/ (TIP: Download the portable version so you only need the .exe and dont need to download.)

  In RUFUS chose GPT or MBR, keep all other setting default and press ready. Close rufus after and eject your drive. 

Go into the bios of your pc/server and chose the usb or other boot media and chose boot order too boot into it. 

2.
When Debian 13 or newer launches chose "Graphical install"

Go through the first couple of steps and fill out your appropriate region, language and keyboard. 

You will need a Ethernet connection to proceed with network setup.

3. 
We will now go though setting up more curcial components.

Hostname: As this setup is for local network use, the naming scheme will follow a easy to follow scheme that matches my other machines on the network. 

Domainname: This is a local network so the domainname is not important, but fill out .("yourowndomain") or whatever else. 

Root password: It is advised to set up if you plan on deplying the server.

Full name: Yes fill out your name

Usermame: Something that matches other machines on the network

Configure timezone is the next step

4. 
Partitioning disk
  If you have encrypted drives this is not necessary. But recommended by many is using either use entire disk and set up LVM or use enitre disk and set up encrypted LVM.
    We are chosing LVM for this setup.

Chose your Operative system drive

You may chose to make seperate partitions, but it is really a hassle and I chose not to. 

Check [yes] for partition disk

Check if it has selected your whole drive in the next step

It will now prompt you accept the partitions it has created

You will now install the BASE system after partitioning

5. 
Next debian will ask where the package manager should download its packages from after installation and for your apt commands.
  Choose your own country or one close by.

...
