# Getting started with RPi

## Load Image onto the microSD Card

1. Insert microSD card into computer (through SD card adapter if necessary)
2. Download and run Raspberry Pi Imager from https://www.raspberrypi.com/software/
3. Select "Raspberry Pi 5" as the device
4. Select "Raspberry Pi OS (64bit)" as the OS
5. Select the SD Card volume as the storage
6. Click "Edit Settings" when asked to apply OS customization
    ### General (notate hostname, username, and password)
    1. Set your hostname
    2. Set your username/password
    3. Configure your wireless connection (LAN country "US")
    4. Set locale
    ### Services
    1. Enable SSH and Allow public-key auth only
    2. Use `ssh-keygen -t rsa` in terminal to create a .pub and key file. When asked, use a path like `/Users/<YOURUSERNAME>/.ssh/pi5`
    3. Then run `cat /Users/<YOURUSERNAME>/.ssh/pi5.pub` to see and copy the .pub file's contents into the text input in the Pi Imager.
7. Click "Yes" to apply OS customization settings and wait ~5 minutes for the card to be written to.
8. Remove card upon write being successful. Insert the card into the Pi and power it on.

## SSHing into the Pi

1. `ssh -i <PATH_TO_PRIVATE_KEY> <USERNAME>@<HOSTNAME>`

## Starting a basic nginx webserver

1. `sudo apt install nginx`
2. `sudo systemctl start nginx` (this will serve on port 80)
3. `sudo systemctl enable nginx`

You can change the port and nginx defaults by doing: `sudo nano /etc/nginx/sites-available/default`

4. `sudo systemctl restart nginx`

The default served folder is `/var/www/html`

# Routing traffic for your own domain with Cloudflare Tunnels

## Set up Tunnels in Cloudflare

1. Sign up for CloudFlare: https://dash.cloudflare.com/sign-up
2. Add a website using your existing domain (select the free plan)
3. Update your name servers with your registrar to Cloudflare's as specified during the domain activation.
4. It may take up to 24 hours for nameservers to update--but usually _much_ quicker. You can push Cloudflare to check again on Cloudflare's website dashboard. Look for a "Check nameservers now" button.
5. Zero Trust -> Create Team Name -> Select and enable the free plan
6. Zero Trust -> Networks -> Tunnels -> Add a tunnel -> "Cloudflared" -> Debian -> arm64-bit -> use the commands specified in the "Install and run a connector" on the Pi. -> You should then see a new Connector ID show up in "Connectors".
7. Setup tunnel routes: Zero Trust -> Networks -> Tunnels -> YOUR_TUNNEL_NAME -> Edit button -> Public Hostname tab -> Set your domain, and your URL to http://localhost:PORT_HERE -> "Save hostname" button.
   If the `cloudflared` service is running properly and connected, changes to the tunnels will be immediate.

# Pi-Hole

## Setup of Pi-Hole

1. `curl -sSL https://install.pi-hole.net | bash`
2. Assign your Pi a static IP address on your network
3. Select the network interface you're using eth0 or wlan0. Select the appropriate one by pressing space bar.
4. Select the defaults for everything else in the install (Google, standard blocklists, admin web interface, etc)
5. Notate the IPv4 address for the Pi-hole, and the web interface address (http://pi.hole/admin), and the **admin password**.
6. `sudo nano /etc/lighttpd/lighttpd.conf`
7. Change server.port from `80` to `8888`
8. `sudo service lighttpd restart`
9. Verify admin access at `http://<HOSTNAME>:8888/admin/login.php`
10. Change device's DNS to the Pi's IP address to test:
    1. Mac Settings -> Wi-Fi -> Details (next to your wifi network) -> DNS -> "+" button -> Type your Pi's IP -> "OK Button"
11. Test here: https://canyoublockit.com/testing/
    Note!! If you can't get online any longer, or websites are broken, change DNS with same directions but hit the "-" button on the IP you added.

You can optionally set the new DNS network-wide with your router. All routers are different, so steps vary :)

# Home Assistant

## Installation of Docker

1. `for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done`
2. `sudo apt-get update`
3. `sudo apt-get install ca-certificates curl`
4. `sudo install -m 0755 -d /etc/apt/keyrings`
5. `sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc`
6. `sudo chmod a+r /etc/apt/keyrings/docker.asc`
7. `echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null`
8. `sudo apt-get update`
9. `sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin`
10. Verify docker is functioning: `sudo docker run hello-world`

## Installation of Home Assistant

1. `mkdir -p ~/home_assistant/config`
2. `sudo docker run -d --name homeassistant --privileged --restart=unless-stopped -e TZ=America/New_York -v ~/home_assistant/config:/config -v /run/dbus:/run/dbus:ro --network=host ghcr.io/home-assistant/home-assistant:stable`

## Setup of Home Assistant

1. Go to http://YOUR_PI_HOSTNAME_OR_IP:8123
2. Click "Create Smart Home" button and follow basic setup
