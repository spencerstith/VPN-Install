# VPN-Install | Spencer Stith

## Sign up for a Digital Ocean account
1. I had trouble doing this in Safari, so Chrome might be better. Go to https://m.do.co/c/4d7f4ff9cfe4
2. Sign up for a FREE (not free) account!!
3. Confirm email.
4. Enter payment information. You will need $5.

## Create an Ubuntu Droplet
1. Form the DigitalOceans homepage, go to "Explore control center" -> Droplets.
2. Create a new Droplet. Choose the basic, default, cheap options. (Ubuntu, Basic, $5/month, Regular Intel CPU).
3. Choose where you want your datacenter to be. I chose New York.
4. Create password (SSH is more secure but I'm not too worried).
5. Choose no additional options.

## Install Docker (What is a container?)
I used the instructinos provided at https://thematrix.dev/install-docker-and-docker-compose-on-ubuntu-20-04/
Here are all the steps I did:
1. Navigate to the console by going to your project, clicking on the droplet, then console.
2. Once you are in, install the necessary tools with `sudo apt install apt-transport-https ca-certificates curl software-properties-common -y`
3. Then, add a Docker key: `curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -`. It should say "OK".
4. Add the docker repo. Here we are using the 32bit/64bit OS: `sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"`
5. Switch to the correct repo: `apt-cache policy docker-ce`
6. Finally, install Docker: `sudo apt install docker-ce -y`. You are root by default so no worries about permissions.
7. Install Docker Compose: `sudo curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose`
8. Set persmissions: `sudo chmod +x /usr/local/bin/docker-compose`

## Install Wireguard
I used the instructions at https://thematrix.dev/setup-wireguard-vpn-server-with-docker/
Here are all the steps I did:
### Setup
1. Run these to create the correct directories and make the compose file:
```
mkdir -p ~/wireguard/
mkdir -p ~/wireguard/config/
nano ~/wireguard/docker-compose.yml
```
2. Paste this into the newly created `docker-compose.yml`:

```
version: '3.8'
services:
  wireguard:
    container_name: wireguard
    image: linuxserver/wireguard
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Chicago
      - SERVERURL=137.184.25.252
      - SERVERPORT=51820
      - PEERS=pc1,pc2,phone1
      - PEERDNS=auto
      - INTERNAL_SUBNET=10.0.0.0
    ports:
      - 51820:51820/udp
    volumes:
      - type: bind
        source: ./config/
        target: /config/
      - type: bind
        source: /lib/modules
        target: /lib/modules
    restart: always
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
```
3. You need to modify a few things, but I went ahead and replaced them in the code above.
4. I changed timezone, `TZ`, to `America/Chicago`
5. You have to change your server URL, `SERVERURL`, which is found super easily on the DigitalOceans dashboard.
6. The last change is the number of user-cofig files to generate. I will just stick with the default number of 3.
7. Start Wireguard:
```
cd ~/wireguard
docker-compose up -d
```

## Testing the VPN
### Testing on a Phone
1. Navigate to the App Store (iPhone in my case) and download the Wireguard app. It should be the first one with a dragon.
2. Before entering the VPN, go to [IP Leak](https://ipleak.net) to see your before IP address. Here is mine:

![IMG_2853](https://user-images.githubusercontent.com/42558850/143940184-a07c2c08-284d-4aa8-bc2a-d5a5a3d8c66b.PNG)

3. In the DigitalOcean console, show the execution log, which will show a QR code to allow you to add this VPN to the app: `docker-compose logs -f wireguard`
4. In the app, tap the `+` button to add a new tunnel, and scan this QR code. Choose the QR code labeled `PEER phone1`
5. On the iPhone, you are going to have to let it have permissions. Just enter your password and follow the prompts it gives.
6. You are now connected! Go back to IPLeak and look at your new IP. Here is mine:

![IMG_2854](https://user-images.githubusercontent.com/42558850/143940249-0a83aedf-fd14-4f0e-ab57-7c9b4954013c.PNG)

Here are my settings inside the Wireguard App:

![IMG_2856](https://user-images.githubusercontent.com/42558850/143940206-cdfab750-24eb-46eb-8e06-3a932590133d.PNG)

### Testing on a Computer
1. Navigate to the App Store (macOS in my case) and download the Wireguard software. It can be found here: https://apps.apple.com/us/app/wireguard/id1451685025?mt=12
2. In the DigitalOceans console window, navigate to your profile for the VPN: `cd ~/wireguard/config/peer_pc1`. In this case we are using `peer_pc1`.
3. You will now need to grab the configureation file. The easiest way is to look at the file, copy the contents, then paste them onto your local machine. Here is how:
4. Open the file: `nano peer_pc1.conf`
5. Copy the contenta of the file with good-old Command-C
6. CTRL + X to exit
7. Open TextEdit on your local machine and paste the contents.
8. Save somewhere safe. I called mine `peer_pc1.conf`, just as it was in the Ubunty machine.
9. Open Wireguard and hit "Add new tunnel" and open that config file we just made.
10. You can now activate the VPN! Here are screen shots of my Wireguard software deactivated/activated and a side-by-side of my before and after IP addresses

Deactivated VPN window
<img width="912" alt="Screen Shot 2021-11-29 at 2 35 56 PM" src="https://user-images.githubusercontent.com/42558850/143940422-105bd0e1-d3db-4812-b580-e3fc59eb8a84.png">

Activated VPN window
<img width="912" alt="Screen Shot 2021-11-29 at 2 36 16 PM" src="https://user-images.githubusercontent.com/42558850/143940482-a216a7b3-da8b-4e26-9a85-66faacb58191.png">

Side-by-side IP addresses
<img width="1440" alt="Screen Shot 2021-11-29 at 2 37 53 PM" src="https://user-images.githubusercontent.com/42558850/143940534-13fe92b4-8b60-40c9-99be-852ef4d07a93.png">


