# Media Server Setup

## Objectives

- To create a secure and remotely-accessible media server
- Make best-effort to use open-source resources
  - [PiVPN](https://github.com/pivpn/pivpn)
  - [Jellyfin](https://jellyfin.org/)

## Specifications

- Implemented on an Intel i5 system with 8GB of installed RAM
- Operating System: Ubuntu Server 20.04.1 LTS
  - Should work on most Debian-based operating systems including Raspbian

## Assumptions

- Operating system is set up with sudo access on a server operating system
- Internet connection with a configured static IP address
- SSH key has been generated on another system to be used to access the server

## Updating the System and Software Installation

- mediainfo: manage the metadata associated with media files
- ffmpeg: codec package for media files
- docker.io: Docker support for containerized operation

```bash
sudo apt-get update && sudo apt-get upgrade
sudo apt-get install mediainfo ffmpeg docker.io
```

## Authentication for Remote Access

- Add generated public keys to the 'authorized keys' directory
- Update the ssh configuration to disallow password authentication
- Update the ssh port

```bash
# Verify that the key has been added
ls .ssh/authorized_keys

# Update the port number and disable password authentication with SSH
sudo sed -i 's/#Port 22/Port 12345/g' /etc/ssh/sshd_config
sudo sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/g' /etc/ssh/sshd_config

# Restart the SSH service
sudo sevrice ssh restart
```

### Securing SSH Access with 2FA

#### Updating PAM

```bash
# Install the Google authentication module
sudo apt install libpam-google-authenticator

# Start the google-authenticator setup and use best judgement for each step
google-authenticator

# Remove the password prompt
sudo sed -i 's/@include common-auth/# @include common-auth/g' /etc/pam.d/sshd

# Add the Google authentication module into PAM
echo 'auth required pam_google_authenticator.so' | sudo tee -a /etc/pam.d/sshd
```

#### Updating SSH

```bash
# Comment out the original ChallengeResponseAuthentication Value
sudo sed -i 's/ChallengeResponseAuthentication/# ChallengeResponseAuthentication/g' /etc/ssh/sshd_config

# Allow for the challenge response to be entered and expect keyboard input during authentication
echo 'ChallengeResponseAuthentication yes' | sudo tee -a /etc/ssh/sshd_config
echo 'AuthenticationMethods publickey,keyboard-interactive' | sudo tee -a /etc/ssh/sshd_config

# Restart SSH service
sudo service ssh restart
```

## Installing and Configuring VPN

- Follow the prompts offered by PiVPN
- Example configuration:
  - Protocol: WireGuard
  - Static IP address
  - DNS Hostname: mymediaserver.example.com (Provided by Dynamic DNS)
  - Port: 54321

```bash
# Load and run the script to set up VPN
curl -L https://install.pivpn.io | sudo bash

# Add a new host certificate to the VPN
pivpn add -n example_user

# Display user QR code to be scanned by phone or device
pivpn -qr example_user
```

## Media Server Container

### Initializing Folders

- Folder locations that will be used for media files passed to the server
  - media: parent folder
  - media/audio: home for all audio files
  - media/video: home for all video files
- Folder location that will be used for the containers data (useful if the container needs to be reconstructed)
  - jellyfin: parent folder
  - jellyfin/config: configurations folder

```bash
mkdir -p media/audio media/video
mkdir -p jellyfin/config
```

### Docker Container Setup

```bash
# Create the Docker user group and add the current user to the group
# Allows the usage of Docker without the need for running with sudo
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
sudo service docker restart

# Confirm that Docker is operational
docker run hello-world

# Pull the image for Jellyfin
docker pull linuxserver/jellyfin

# Run the server with the following options (update as desired)
# Name: jellyfin
# Port to be mapped to Jellyfin's internal port 8096: 9876
# Volume for media: media/
# Volume for configurations: jellyfin/config/
# Restart: unless-stopped (will restart on boot and docker service restert)
# Image (do not change): linuxserver/jellyfin
docker run -d \
--name=jellyfin \
-p 9876:8096 \
-v "$PWD/media:/data/media" \
-v "$PWD/jellyfin/config:/config" \
--restart unless-stopped \
linuxserver/jellyfin
```

## Optional System Updates

- The above system can be intalled on many kinds of systems including laptops
  - Because of this, a couple of configurations should be made including:
    - Disable wifi radio
    - Disable any system actions caused by the lid being closed
    - Battery may be removed if system hardware allows for running directly from external power

```bash
# Install the network-manager
sudo apt-get install network-manager

# Disable the wifi radio
nmcli r wifi off

# Edit the following Login Daemon to ignore the lid switch  
sudo nano /etc/systemd/logind.conf
# Uncomment and update: HandleLidSwitch=ignore
# Uncomment and update: HandleLidSwitchExternalPower=ignore
# Uncommant and update: HandleLidSwitchDocked=ignore

# Restart the login daemon
sudo service systemd-logind restart
```
