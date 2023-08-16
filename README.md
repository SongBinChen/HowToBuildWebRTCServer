# Build WebRTC Server


## WebRTC Introduction

WebRTC Demo project: https://github.com/webrtc/apprtc

## Installation
### Environment

1. Server is setup on Tencent Cloud with Ubuntu18.04.
2. Open common ports(3478、8080、8089、80、443) on cloud server firewall.
3. Software:git、tar、wget、unzip
4. Development tools:NodeJS、Python、Golang
   
Install Common Tools:
```
sudo apt-get update 
sudo apt-get upgrade
sudo apt install -y nodejs npm python golang-go unzip git-core tar wget openjdk-8-jdk
```

Install **node.js**:
```
sudo apt install nodejs
# check version
node --version

sudo apt-get install npm 
npm --version

# install gnurt
npm -g install grunt-cli@1.3.2
grunt --version

# install coffeescript
npm install --dev coffeescript
```

### Dependence Service
Install **google_appengine**:
```
mkdir webrtc
cd webrtc
wget https://storage.googleapis.com/appengine-sdks/featured/google_appengine_1.9.40.zip
unzip google_appengine_1.9.40.zip
echo "export PATH=$PATH:/home/user_name/webrtc/google_appengine" >> ~/.zshrc
```
Install **libevent**:
```
wget https://github.com/downloads/libevent/libevent/libevent-2.0.21-stable.tar.gz
tar xvf libevent-2.0.21-stable.tar.gz
cd libevent-2.0.21-stable
./configure
make
sudo make install
```

### Signal Service


### STUN/TURN Service

### WebRTC Client Service

## Configuration

### ICE 

### Signal Service

### Go directory

## Certification

