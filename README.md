# NaturewatchCameraServer

This is a Python server script that captures a video stream from a Pi Camera and serves it as a .mjpg through a control website to another device. Part of the My Naturewatch project in collaboration with the RCA.

This readme describes a modified installation procedure to get this running on Docker on Raspbian Buster

## Enable SSH (to be automated) 

Add a blank file named `ssh` to the boot folder on the SD card

## Set up OTG ethernet (to be automated) 

Follow the guide created by gbaman to set up OTG ethernet over USB serial https://gist.github.com/gbaman/975e2db164b3ca2b51ae11e45e8fd40a

## Requirements

### Install some required packages first
	sudo apt update
	sudo apt install -y \
	     apt-transport-https \
	     ca-certificates \
	     curl \
	     gnupg2 \
	     software-properties-common

### Get the Docker signing key for packages
	curl -fsSL https://download.docker.com/linux/$(. /etc/os-release; echo "$ID")/gpg | sudo apt-key add -

### Add the Docker official repo
	echo "deb [arch=armhf] https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \
     		$(lsb_release -cs) stable" | \
    		sudo tee /etc/apt/sources.list.d/docker.list

### Install Docker
The aufs-package, part of the "recommended" packages, won't install on Buster just yet, because of missing pre-compiled kernel modules. Apparently, aufs is *not* a requirement, just a recommendation and it is debatable whether it should be. 

Install Docker without the recommended packages by using the "--no-install-recommends" switch

	sudo apt update
	sudo apt install -y --no-install-recommends docker-ce

### Enable Docker
	sudo systemctl enable docker
	sudo systemctl start docker

## Install Docker Compose
Some people recommend installing docker-compose via pip, but I had issues with that way. Modules weren't found. I didn't dig deeper since I found
	sudo apt install docker-compose
does the trick. Test it with:
	docker-compose --version

## Make sure the user has the neccessary rights

Add pi user to docker group and the video group

	sudo usermod -aG docker $USER
	sudo usermod -aG video $USER


## Running the server

Build the docker container
	
	docker-compose build
	
## Configuring the wifi setup

Run the config setup python script. This will reboot the pi
	
	sudo python3 NaturewatchCameraServer/cfgsetup.py

## Access the interface
    
The website is then accessible through its hostname:

	http://raspberrypi.local/
	
Be sure to replace `raspberrypi.local` with whatever hostname the Pi has.

## Running tests

You can run tests directly on the Raspberry pi to test the various functions of the
software as well as the API. After building the container, run the tests with pytest.

    docker run \
    --device /dev/vcsm --device /dev/vchiq \
    -p 5000:5000 \
    -v ~/data:/naturewatch_camera_server/static/data \
    naturewatchcameraserver \
    pytest -v naturewatch_camera_server/

