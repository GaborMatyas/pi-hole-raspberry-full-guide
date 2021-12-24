# Install and use PI-hole with Raspberry.md
## Setup a Raspberry PI 3 or 4 with Raspberry PI OS 64bit
### Choosing the right version of Raspberry Pi OS


You can download the images for Raspberry Pi OS from the official website.

As of writing, images for the 64-bit variant are still in beta and can be found [here](https://forums.raspberrypi.com/viewtopic.php?f=117&t=275370). Its lite version without GUI is [available](https://downloads.raspberrypi.org/raspios_lite_arm64/images/raspios_lite_arm64-2021-11-08/) too.

Once you have downloaded the IMG file you can use the [Raspberry Pi Imager](https://www.raspberrypi.com/software/) to write the operating system in a SD card.

As for which image to choose, Docker works on all variants and editions of Raspberry Pi OS.
32-bit vs 64-bit

The 32-bit edition of Raspberry Pi OS will run on every board, including Raspberry Pi 2.

If you have a Raspberry Pi 3, 4, or 400, as well as the Raspberry Pi Zero 2, you have the opportunity to pick the 64-bit variant too. Using a 64-bit operating system will give you better performance and it’s required to take advantage of the full amount of memory of the 8GB Raspberry Pi 4 board.

What should you choose, Desktop vs Lite?
As the names suggest, the Desktop edition comes with a graphical user interface and the ability to run desktop apps. The Lite edition is headless, offering only access to the command line.

Both editions of Raspberry Pi OS can run Docker.

If you plan to use your Raspberry Pi as a headless server, pick the Lite edition to save disk space and reduce memory usage (and have a smaller potential attack surface). You will be able to control the server remotely via SSH.

[Useful tutorial from youtube to Install & Set Up Raspberry Pi OS - Pi4 Pi3 Pi2](https://www.youtube.com/watch?v=y45hsd2AOpw)

## Login to Raspberry
When the lite version has been started, just type in a command like 'cls' to delete your screen and the PI will require login credentials. The default ones at the time of writing:
username: pi
password: raspberry

## How To Change the Password
Here's how to change your password on Raspberry Pi OS:
When logged in, open a Terminal window. Enter the passwd command and press Enter. The system will prompt you to confirm your current password. After this, enter your new password twice, pressing Enter after each entry. As is the Linux standard, you won't see any characters on screen while entering this password.

If everything worked properly, you'll see a password updated successfully message. This means that you're good to go with the new password. Make sure you remember it!

For reference, if you ever need to remove the password from an account on your Raspberry Pi, you can use the command sudo passwd [USERNAME] -d. Of course, it's wise to keep a password on all accounts.

## Enable and use SSH on Raspberry Pi in the Terminal
Open the terminal on your Raspberry Pi and run the tool by typing:
```
sudo raspi-config.
```
Use the arrows on your keyboard to select Interfacing Options.
Select the P2 SSH option on the list.
Select Yes on the “Would you like the SSH server to be enabled?” prompt.
Get the IP address of your PI with `ifconfig` command and type it to `Putty application` on Windows to connect to your PI.

---

## Setup Docker
### Install some required packages first
```
sudo apt update
sudo apt install -y \
     apt-transport-https \
     ca-certificates \
     curl \
     gnupg2 \
     software-properties-common
```

### Install Docker
```
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
```

To allow non-root users to use Docker after the installation is complete, execute the following command (such as the default 'pi' user on Raspberry Pi OS).
```
sudo usermod -aG docker pi
```

Start Docker automatically when your Raspberry Pi reboots by running
```
sudo systemctl enable docker
```

If we try to start docker service when docker is masked, we might face the error ‘Failed to start docker.service. So run this command.
```
sudo systemctl unmask docker
```

Once the docker unit is unmasked, we can start the docker daemon with the systemctl command.
```
systemctl start docker
```

To verify whether the docker service is active and running. We will use the systemctl status command, which shows the current status of the particular service. Execute the command below on your Terminal.
```
systemctl status docker
```

---

## PiHole
- [Piholw Docker Hub page](https://hub.docker.com/r/pihole/pihole)
- [The list of Timezones what you can set up in pihole-docker conainers](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)
- [PiHole Docker Install documentation](https://github.com/pi-hole/docker-pi-hole/#running-pi-hole-docker)

### PiHole docker container
Use the following commands to create the docker-compose.yaml file:
```
mkdir pihole
cd pihole
nano docker-compose.yaml
```

Then paste the following content in the file and save it.
It is not mandatory to setup a password here, but it is way more easier than anywhere else, so use the commented out 'WEBPASSWORD' property in the yaml file.

```yaml
version: "3"

# More info at https://github.com/pi-hole/docker-pi-hole/ and https://docs.pi-hole.net/
services:
  pihole:
    container_name: pihole
    image: pihole/pihole:latest
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "67:67/udp"
      - "80:80/tcp"
      - "443:443/tcp"
      - "8080:80"
    environment:
      TZ: 'Europe/Budapest'
      # WEBPASSWORD: 'set a secure password here or it will be random'
    # Volumes store your data between container upgrades
    volumes:
      - './etc-pihole:/etc/pihole'
      - './etc-dnsmasq.d:/etc/dnsmasq.d'
    # Recommended but not required (DHCP needs NET_ADMIN)
    #   https://github.com/pi-hole/docker-pi-hole#note-on-capabilities
    restart: unless-stopped
    privileged: true

```

### Install Docker Compose
```
sudo apt-get install libffi-dev libssl-dev
sudo apt install python3-dev
sudo apt-get install -y python3 python3-pip
sudo pip3 install docker-compose
```

### Add your user to docker group
```
sudo gpasswd -a $USER docker
newgrp docker
```

If you have a problem, you may try after logging out and login back, or reboot. Or simply do:
```
sudo su $USER
```

Run docker-compose up -d to build and start pi-hole
```
docker-compose up -d
```

### Add further blocklist to your PiHole
[Youtube video tutorial](https://youtu.be/0wpn3rXTe0g?t=238)

- Login to the PIHole admin panel in your browser by typing in the following URL: "<ip_address_of_the_raspberry>/admin".
- Login with your PiHole password.
- Navigate to `Group Management / Adlists` menu.
- Use [the currated list to expand your blocklist](https://firebog.net/).
- Copy a link like `https://raw.githubusercontent.com/PolishFiltersTeam/KADhosts/master/KADhosts.txt` from the previous page and paste it in to your PiHole's Add a new list section's "Address" input field.
- Add a comment like 'firebog1' or anything that helps to filter your list later and click on ADD button.
- Make sure everyone of them enabled
- Go to `Tools / Update gravity` and click on "Update".
- This will pulls down all the information from these lists and add them to your PiHole blocklist. ...it can take a while...

### Update pihole to its new version when it is already running
[PiHole update when using Docker](https://github.com/pi-hole/docker-pi-hole#upgrading-persistence-and-customizations)
Use the following commands:
```
docker pull pihole/pihole

docker-compose down

docker-compose up -d
```

## Turn off your Raspberry PI from terminal when necessary
Use the following command:
```
sudo shutdown -h now
```
