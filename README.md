# Build WebRTC Server


## WebRTC Introduction

WebRTC Demo project: https://github.com/webrtc/apprtc

## Installation
### Environment

1. Server is setup on cloud service with Ubuntu18.04.
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
# local IP
echo "export LOCAL_IP=xx.xx.xx.xx" >> ~/.zshrc
# external IP you can reach outside
echo "export SERVER_IP=xx.xx.xx.xx" >> ~/.zshrc
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

### Nginx
Note:If your cloud server already has nginx service, skip this part.
```
# Download nginx
wgwt http://nginx.org/download/nginx-1.16.1.tar.gz
tar xvf nginx-1.16.1.tar.gz
# Install PCRE
apt-get install libpcre3-dev
cd nginx-1.16.1
./configure --with-http_ssl_module
make
sudo make install
```

### Build AppRTC
```
cd /home/user_name/webrtc/apprtc
npm install --no-fund
grunt build
```

## Configuration and Run Service

### Turn Server
Set the username and password on coturn Nat. These will be used in apprtc stun/turn configuration.
```
sudo nohup turnserver -L $LOCAL_IP -a -u user:password -v -f -r nort.gov &

## Check by lsof -i:3478
#COMMAND     PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
#turnserve 14754 user   16u  IPv4 112232      0t0  UDP instance-5i00bk4l:3478 
#turnserve 14754 user   17u  IPv4 112233      0t0  UDP instance-5i00bk4l:3478 
#turnserve 14754 user   32u  IPv4 112263      0t0  TCP instance-5i00bk4l:3478 (LISTEN)
#turnserve 14754 user   33u  IPv4 112267      0t0  TCP instance-5i00bk4l:3478 (LISTEN)
```

### Collider Signal Server
Modify certification file location. Configure file is `apprtc/src/collider/collider/collider.go`
```
## Search and change the file here
e = server.ListenAndServeTLS("/CustomizedDirectory/xxxx.crt", "/CustomizedDirectory/xxxx.key")
```

Modify domain. Configure file is `apprtc/src/collider/collidermain/main.go`
```
## Search and change the room server domain name
var roomSrv = flag.String("room-server", "https://www.domainname.com", "The origin of the room server")
```

Rebuild collider
```
go get collidermain
go install collidermain
```

Run collider server
```
nohup $GOPATH/bin/collidermain -port=8089 -tls=true  -room-server="https://$SERVER_IP:8080" &
```

### AppRTC
Apprtc configure file is `apprtc/src/app_engine/constants.py`

Change **stun/turn** server:
```
ICE_SERVER_OVERRIDE  = [
   {
     "urls": [
       "turn:your external ip:3478?transport=udp",
       "turn:your external ip:3478?transport=tcp"
     ],
     "username": "coturn Nat user",
     "credential": "coturn Nat password"
   },
   {
     "urls": [
       "stun:your external ip:3478"
     ]
   }
 ]
```

Change **ICE** and **WSS** configuration:
```
ICE_SERVER_BASE_URL = 'https://www.domainname.com'
...
 # Dictionary keys in the collider instance info constant.
 **WSS_INSTANCE_HOST_KEY = 'www.domainname.com:8089' // Apply from cloud service provider
 WSS_INSTANCE_NAME_KEY = 'vm_name'
 WSS_INSTANCE_ZONE_KEY = 'zone'
 WSS_INSTANCES = [{
     WSS_INSTANCE_HOST_KEY: 'apprtc-ws.webrtc.org:443',
     WSS_INSTANCE_HOST_KEY: 'www.domainname.com:8089', // Apply from cloud service provider
     WSS_INSTANCE_NAME_KEY: 'wsserver-std',
     WSS_INSTANCE_ZONE_KEY: 'us-central1-a'
 }, {
     WSS_INSTANCE_HOST_KEY: 'apprtc-ws-2.webrtc.org:443',
     WSS_INSTANCE_HOST_KEY: 'www.domainname.com:8089', // Apply from cloud service provider
     WSS_INSTANCE_NAME_KEY: 'wsserver-std-2',
     WSS_INSTANCE_ZONE_KEY: 'us-central1-f'
 }]
```
Compile apprtc and run:
```
npm install --no-fund
grunt build
## run
sudo nohup dev_appserver.py --host=$LOCAL_IP /user/apprtc/out/app_engine --skip_sdk_update_check &

```
### Nginx
Modify the configuation file. `/etc/nginx/nginx.conf`
```
http {

	##
	# Basic Settings
	##

	sendfile on;
	tcp_nopush on;
	tcp_nodelay on;
	keepalive_timeout 65;
	types_hash_max_size 2048;

	include /etc/nginx/mime.types;
	default_type application/octet-stream;

	##
	# SSL Settings
	##

	ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
	ssl_prefer_server_ciphers on;

	##
	# Logging Settings
	##

	access_log /var/log/nginx/access.log;
	error_log /var/log/nginx/error.log;

	##
	# Gzip Settings
	##

	gzip on;

	##
	# Virtual Host Configs
	##

	include /etc/nginx/conf.d/*.conf;
	include /etc/nginx/sites-enabled/*;
    upstream roomserver {
         ## your cloud server IP
        server external IP:8080;
    }
    server {
     root /usr/share/nginx/html;
     index index.php index.html index.htm;
     #SSL default port 443
     listen 443;
     ssl on;
     # server which is bind ssl cert
     server_name www.domainname.com;
     # your ssl cert
     ssl_certificate  xxxx.crt;
     # you ssl key
     ssl_certificate_key xxxx.key;
     ssl_session_timeout 5m;
     ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
     ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
     ssl_prefer_server_ciphers on;
     charset utf-8;
     location / {
       proxy_pass http://roomserver$request_uri;
       proxy_set_header Host $host;
     }
    }
    server {
     listen 80;
     # server name which is bind ssl cert
     server_name www.domainname.com;
     # domain request change from http to https
     return 301 https://roomserver$request_uri;
    }
}
```
Run nginx:
```
# check the configure error
nginx -t
nginx -s reload
```


## Certification


## Test
Open the website `www.domainname.com`

## Troubleshooting
### Start AppRTC
Error:
```
ImportError: No module named _sqlite3
```
Switch to pytho2.7.9 below and add param when configure:
```
./configure --enable-loadable-sqlite-extensions
```
### Start nginx
```
nginx: [error] invalid PID number "" in "/run/nginx.pid"
```
Relocate nginx confiure file:
```
 sudo nginx -c /etc/nginx/nginx.conf
 sudo nginx -s reload
```



## Reference
https://www.freesion.com/article/39421331363/  
https://developer.aliyun.com/article/877670  
https://www.jianshu.com/p/df300071d8d6  
https://blog.csdn.net/paradox_1_0/article/details/108707731  
https://juejin.cn/post/6844904119807770637  

