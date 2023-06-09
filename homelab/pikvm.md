# pikvm setup with tailscale+https
i often need remote access to my home pc, and while [i do use other methods of remote access](./README.md), this is the only way i can access the bios or wake my pc (havent gotten wol to work)

## materials
- Raspberry Pi 4b
- SD Card
- USB HDMI capture card
- Various Cables
  - USB C male -> USB A male
  - HDMI male -> male
- Tailscale Account

## Prerequisites
- basic knowledge of the linux command line
- basic knowledge of [tailscale](https://tailscale.com/kb/start/)

## Steps
1. [download pikvm](https://pikvm.org/download/) (i have a usb capture card so i selected the HDMI-to-USB option)
2. open downloaded file with [raspberry pi imager](https://www.raspberrypi.com/software/) and flash onto sd card
   1. (ONLY IF USING WIFI) mount the flashed sd card again, edit `pikvm.txt` and add 
      ```
      FIRSTBOOT=1
      WIFI_ESSID='mynet'
      WIFI_PASSWD='p@s$$w0rd'
      ```
      to it, then unmount the sd card 
3. insert sd card into pi
4. connect cables (see image for reference)
   1. connect pc's hdmi output into capture card
   2. connect capture card into pi (bottom USB2 port, i have an extension for convenience)
   3. connect usb c into pi's usb c port, and the other end into pc's usb a port
   <img src="./media/pikvm.jpg" height="500">
5. login to raspberry pi via SSH or physically (default username is `root` and password is `root`, i highly suggest [changing it](https://docs.pikvm.org/first_steps/#first-power-on))
6. [download/run tailscale on the pikvm](https://docs.pikvm.org/tailscale/)
   ```
   # rw
   # pacman -Syu tailscale-pikvm
   # systemctl enable --now tailscaled
   # tailscale up
   ```
7. [setup tailscale https](https://tailscale.com/kb/1153/enabling-https/)
8. generate https keys for pikvm
   1. on the pikvm, navigate to `/etc/kvmd/nginx/ssl/`
   2. there should be a server.key and server.crt file already, delete or rename them
   3. run `tailscale cert` without arguments, it'll give you a domain to use, then run `tailscale cert <domain>` again
   4. rename the two certificates generated by tailscale to `server.key` and `server.crt`
9. edit config files to allow waking PC via USB
   1.  make sure your PC allows waking via USB in the BIOS
   2.  update system with `pacman -Syu`
   3.  edit `/etc/kvmd/override.yaml` and add
        ```
        otg:
        remote_wakeup: true
        ```
        to the bottom of the file

10. in another device connected to your tailnet, navigate to your pikvm's tailscale domain