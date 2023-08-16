# Build WebRTC Server


## WebRTC Introduction

WebRTC Demo project: https://github.com/webrtc/apprtc

## Installation
### Environment

1. Server is setup on Tencent Cloud with Ubuntu18.04.
2. Open common ports(3478、8080、8089、80、443) on cloud server firewall.
3. Software:git、tar、wget、unzip
4. Development tools:NodeJS、Python、Golang
   
Common Tools:
```
sudo apt-get update 
sudo apt-get upgrade
sudo apt install -y nodejs npm python golang-go unzip git-core tar wget openjdk-8-jdk
```

**node.js**:
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
**google_appengine**:
```
mkdir webrtc
cd webrtc
wget https://storage.googleapis.com/appengine-sdks/featured/google_appengine_1.9.40.zip
unzip google_appengine_1.9.40.zip
echo "export PATH=$PATH:/home/user_name/webrtc/google_appengine" >> ~/.zshrc
```
**libevent**:
```
wget https://github.com/downloads/libevent/libevent/libevent-2.0.21-stable.tar.gz
tar xvf libevent-2.0.21-stable.tar.gz
cd libevent-2.0.21-stable
./configure
make
sudo make install
```

### Signal Service
```
#create go directory
mkdir -p /home/user_name/webrtc/goWorkspace/src
echo "export GOPATH=/home/user_name/webrtc/goWorkspace" >> ~/.zshrc
source  ~/.zshrc
cd /home/user_name/webrtc
git clone git://github.com/webrtc/apprtc.git
ln -s /home/user_name/webrtc/apprtc/src/collider/collider $GOPATH/src
ln -s /home/user_name/webrtc/apprtc/src/collider/collidermain $GOPATH/src
ln -s /home/user_name/webrtc/apprtc/src/collider/collidertest $GOPATH/src
go get collidermain
go install collidermain
```

### STUN/TURN Service
```
cd /home/user_name/webrtc
wget http://coturn.net/turnserver/v4.5.0.7/turnserver-4.5.0.7.tar.gz
tar xvfz turnserver-4.5.0.7.tar.gz
cd turnserver-4.5.0.7
./configure
make
sudo make install
```

### Build AppRTC
```
cd /home/user_name/webrtc/apprtc
npm install --no-fund
grunt build
```

## Configuration

### ICE 

### Signal Service

## Certification

## Run Service
### Nginx

## Test

## Troubleshooting

## Reference
