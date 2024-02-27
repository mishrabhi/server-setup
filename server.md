# Server Side Stuff

## ssh

One of the most secure way of login available right now. It creates 2 keys - public and private. Public stays on server and private with us.

Generate ssh key:

```
cd ~/.ssh
ssh-keygen
```

Add ssh to agent:

````
vi ~/.ssh/config

Host*
    AddKeysToAgent   Yes
    useKeyChain    Yes
    ```
````

Add private key to ssh:

```
ssh-add -K ~/.ssh/fsfe
```

Login to Server:

```
ssh root@ip

# use particular ssh key

ssh -i mysshfile root@ip

# debug issues connecting

ssh -i mysshfile root@ip -v
```

## Server Setup:

- Update Software
- Create a new user(don't use root)
- Make that user a super user
- Enable login for that user
- Disable root login

After logged in to the server check whether you are logged in as root by using command `whoami`

## Update Software

```
# update software
apt update

# upgrade software
apt upgrade

# Add new user
adduser $USERNAME
```

## Add user and give permissions

```
addUser $USERNAME

# Add user to sudo group

user -aG sudo $USERNAME

# switch user

su $USERNAME

# check sudo access (important command to check whats happeing on server,like who's trying to login)

sudo cat /var/log/auth.log

# switch to home directory

cd ~

# create new directory if it doesn't exist

mkdir -p ~/.ssh

# login to server

ssh -i $sshPrivateFile $USERNAME@Ip

# create file permission so it will be readable by users

chmod 644 ~/.ssh/authorized_keys

# Disable root login

sudo vi /etc/ssh/sshd_config

# modify PermitRootLogin to no

# restart ssh deamon (sshd) service

sudo service sshd start
```

## Nginx (engine-x)

- Reverse Proxy
- Web Server
- Proxy Server

```
# Install Nginx

sudo apt install nginx

# Start nginx

sudo service nginx start

# Show Nginx Configuration

sudo vi /etc/nginx/sites-available/default

# nginx root directory

/var/www/html

# Change ownership to User (so that we dont have to sudo evrytime we want to perform task in this dir)

sudo chown -R $USER: $USER /var/www

# create applicatin dir

mkdir /var/www/app

# move into app directory and initialise empty git repo

cd /var/www/app && git init
```

## Setup NodeJS

```
Install nodejs and npm

sudo apt install nodejs npm

# install git
sudo apt install git

# Upgrade Node.js

# Download setup script from nodesource

# Run script
sudo bash nodesource_setup.sh

# Install Nodejs
sudo apt install nodejs
```

## Nginx Proxy_pass

Make nginx to route the request to node app running on port 3000

```
sudo vi /etc/nginx/sites-available/default

# Add modify

location {
    proxy_pass 127.0.0.1:3000
}

# Redirects
location /help {
    return 301 https://developer.mozilla.org
}

# Subdomain

server {
    listen 80;
    listen [::]80;   # IPV6 notation

    server_name

    location / {
        proxy_pass 127.0.0.1:3000
    }
}

# Nginx Gzip
/etc/nginx/nginx.conf

# Gzip Setting

gzip.on;
dzip_disable "msie6";

# gzip_vary on;
# gzip_proxied any;
# gzip_comp_level 6;
# gzip_buffers 16 8k;
# gzip_http_version 1.1
```

## PM2 Setup:

```
pm2 start app.js
pm2 start id
pm2 stop id
pm2 log

# set pm2 to start on server restart

pm2 unstartup systemd
pm2 save
```

## More Stuff for Production Server

Install Fail2ban, blocks who tries to login on your server more than 5-10 times.

### Secuirity Checks

- ssh
- Firewalls
- Updates
- Two factor authentication
- VPN

### Install unattended upgrades

```
#Install unattended upgrades
sudo apt install unattended-upgrades

# view conf file
cat /etc/apt/apt.conf.d/50unattended-upgrades
```

### Checking Ports

```
# install nmap (port scanner);
sudo apt install nmap

# Run nmap
nmap YOUR_SERVER_IP_ADDRESS

# Run nmap with more service/version info

nmap -sV YOUR_SERVER_IP_ADDRESS
```

### ufw(uncomplicated firewall)

```
# check firewall status
sudo ufw status

# Enable ufw
sudo ufw enable

# enable ssh
sudo ufw allow ssh

# allow,deny or reject service connection like ssh,http,https

ufw allow http
ufw reject http
ufw deny http
```

### Add http2

```
sudo vi /etc/nginx/sites-available/default

# add
listen 443 http2 ssl;
```

## Checking Running Process

```
# Install htop
sudo apt install htop

# Display running process
htop
```

## Setup PM2

```
# Install pm2
sudo npm i pm2 -g

# Setup ecosystem.config.js

module.exports = {
    apps: [
        {
            name: "app name",
            script: "index.js",
            watch: true,
            cwd: 'pass directory path',
            //environment variables
            env: {
                "PORT": 3100,
                "DB_URI": 'local',
                "NODE_ENV": "development"
            },
            env_production: {
                "PORT": 3200,
                "NODE_ENV": "production"
            }
        },
        {
            name: "second app",
            cwd: 'pass directory path',
            script: "./scripts/start.js",
            watch: true,
            env: {
                "NODE_ENV": "development"
            },
            env_production: {
                "NODE_ENV": "production"
            }

        }
    ]
}

# pm2 commands
pm2 start ecosystem.config.js
pm2 status
pm2 log
pm2 stop appId
```

## Setup MongoDB

```
# Adding the MongoDB Reposiitory

sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0c49F3730359A14518585931BC711F9BA15703C6

echo "deb [ arch=amd64,arm64 ] http://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list

sudo apt-get update

# Installing MongoDB

sudo apt-get install mongodb-org

sudo systemctl start mongod

sudo systemctl status mongod

sudo sysytemctl enable mongod
```

## Securing DB

```
# Adding an Administrative User

use admin
db.createUser {
    {
        user: "adminabhi",
        pwd: "AdminAbhiSecurePassword",
        roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
    }
}

{
    user: "cohousing",
    pwd: "cohousingdev3436",
    roles: [ { role: "readWrite", db: "cohousingdev" } ]
}

# Enabling Authentication

sudo vi /etc/mongod.conf

# uncomment secuirity section (make sure to have a space b/w authorisation: and enabled)

security:
    authorization: enabled

# save the file and quit
sudo systemctl restart mongod

sudo systemctl status mongod

# verifying that Unauthenticated Users are Restricted

mongo -u adminabhi -p --authenticationDatabase admin

# Connection string will be
mongodb://adminashu:AdminAshuSecurePassword@<host/IP>:27017/dbname?authSource=admin
```

## Configure Remote Access

````
sudo ufw status

# if not enabled

sudo ufw enable

# Make sure to allow OpenSSH otherwise you'll be locked out

sudo ufw allow OpenSSH

sudo ufw allow from ```client_ip_address``` to any port ```27017```
````

## Configuring a Public blindIP

````
sudo vi /etc/mongod.conf

# In the net stanza, add the MongoHost's IP to the blindIP line:

net:
    port: 27017
    blindIP: 127.0.0.1, ```IP_of_MongoHost```

sudo systemctl restart mongod

#check local
mongo --username cohousing --password --authenticationDayabase admin --host 13.235.139.232 --port 27017
````

## Clone Template

```
wget --recursive --noclobber --page-requisites --html-extensions --covert-links --restrict-file-names=windows --domains website.org --no-parent www.website.org/tutorial/html/
```
