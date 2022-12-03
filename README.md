# 2420-assign-two

# Author

Sethavan Sen

# Load Balancer

**My Load Balancer** [http://146.190.15.69/](http://146.190.15.69/)

## Creating Digital Ocean Infrastructure
### Note: {} will be used as a filler, this is where you type in your information

### Creating A VPC

1. Visit The Digital Ocean Website and Log in
2. Click on **Networking** located on the left side of the home page
3. Click **VPC** and **Create VPC Network**

![Creating A VPC](/images/1.png)

4. Choose a datacenter region
5. Type a name for your VPC and click create VPC Network

![Creating A VPC](/images/2.png)

5. You have successfully created a VPC!

![Creating A VPC](/images/3.png)

### Creating 2 New Droplets

**Note:** Create an SSH key and add to digital ocean before moving on

1. Create droplet
2. In the setup section, under VPC network, select the vpc you just created

![Creating Droplet](/images/4.png)

3. Choose the SSH key that you created before this step

![Creating Droplet](/images/5.png)

4. Under Finalize and create, select 2 droplets and rename your host name to your liking

![Creating Droplet](/images/6.png)

5. Under Add tags, type web and click create droplet. The Web tage will be used for the load balancer and firewall.

![Creating Droplet](/images/7.png)

6. You have successfully created your droplets

![Creating Droplet](/images/8.png)

### Setting Up Load Balancer

1. In the Manage dropbox click Networking > Load Balancers > Create Load Balancer
2. Under Choose Load Balancer, Select the region where you created your droplets in the previous steps

![Setting Load Balancer](/images/9.png)

3. Under Select VPC Network, select the VPC you created previously

![Setting Load Balancer](/images/10.png)

4. Under Connect Droplets, select web (tag you created previously)

![Setting Load Balancer](/images/11.png)

5. In the select project option, select the project you want the load balancer to belong to and click Create Load Balancer
6. You've created a load balancer!

**Note:** If the status is down, that is okay as a server not installed yet

![Setting Load Balancer](/images/12.png)

### Creating A Firewall

1. In the Manage dropbox, click Networking > Firewalls > Create Firewall
2. Create a name for the firewall
3. Under the Type, select a new option for HTTP and add the load balancer you created into the sources

![Creating Firewall](/images/13.png)
 
 4. Under Apply to Droplets, type web (tag you created previously) and click Create Firewall

You have successfully created a Digital Ocean infrastructure! Now We can move on to the next step

## Creating A New User

**Note:** This step will be done on both your droplets that you have created

1. ssh into your server by running `ssh -i {path/of/server/key} root@{droplet IP}`

![Creating User](/images/14.png)

3. Add a new user by running `useradd -ms /bin/bash {user}`
4. Give your user sudo permissions by running `usermod -aG sudo {user}`
5. Set a password by running `passwd {user}`
6. 5. Run `rsync --archive --chown={user}:{user} ~/.ssh /home/{user}``

![Creating User](/images/15.png)

6. Run `exit`
7. ssh into the new user you just created by running `ssh -i {path/of/server/key} {user}@{droplet IP}`
8. set root login to no by running `sudo vim /etc/ssh/sshd_config` and change **PermitRootLogin to no**

![Creating User](/images/16.png)

9. To apply the settings, run `sudo systemctl restart ssh`
10. Check any update by running `sudo apt update && sudo apt upgrade`

**Repeat with your other droplet**

You have successfully created a new user!

## Installing A Web Server

### Installing Caddy

**Note:** This is done on both droplets

1. Install Caddy by running `wget https://github.com/caddyserver/caddy/releases/download/v2.6.2/caddy_2.6.2_linux_amd64.tar.gz`

![Installing Web Server](/images/17.png)

2. Extract the contents by running `tar xvf caddy_2.6.2_linux_amd64.tar.gz`

You should have the four files shown below when you run `ls`

![Installing Web Server](/images/18.png)

3. Change the ownership by running `sudo chown root: caddy`
4. Copy the file to /usr/bin by running `sudo cp caddy /usr/bin/`

![Installing Web Server](/images/19.png)

You've now created a web server!

## Creating A Web App

**The Next Steps will be done on your wsl local machine**

### Creating Directories

1. Create a main directory to put all your files in by running `mkdir {directory name}`

2. Inside the directory, create 2 new directories by running `mkdir /{previous directory name}/src/ && mkdir /{previous directory name}/html/`

![Creating Web App](/images/20.png)

### Creating An HTML Document

1. Inside the html directory, create and edit an index.html file by running `vim index.html`
2. In the index.html file, write a working simple html document

**Note** I will be changing this html document slightly to differentiate the two servers

![Creating Web App](/images/21.png)

### Creating A Node Project

1. Inside the src directory, create a new node project by running `npm init && npm install fastify`
2. Add the following code to the directory by running vim `index.js`

```
// Require the framework and instantiate it
const fastify = require('fastify')({ logger: true })

// Declare a route
fastify.get('/api', async (request, reply) => {
  return { hello: 'Server x' }
})

// Run the server!
const start = async () => {
  try {
    await fastify.listen({ port: 5050 })
  } catch (err) {
    fastify.log.error(err)
    process.exit(1)
  }
}
start()
```

### Testing If It Works

1. In the scr directory, run `curl https://get.volta.sh | bash`
2. Run `source ~/.bashrc`
3. Run `volta install node`
4. Run `node index.js`

If everything is done correctly, it should look like this!

![Creating Web App](/images/22.png)

### Transfering Files To The Droplet

1. Transfer the directory with the html and src file to both droplets by running `rsync -auvz -e "rsync -auvz -e "ssh -i {path/of/the/key}" {directory/of/files/on/host} {user}@{IP address}:{directory/of/web_one/server}`
2. Move the files to the correct location by running `sudo mv html/ /var/www` and `sudo mc src/ /var/www` on both your droplets


![Creating Web App](/images/23.png)

**Make sure this is done on both your droplets**

## Writing A Caddyfile Server Block

**Note:** This is also done on your local wsl machine

1. Create a file Caddyfile by running `vim Caddyfile` and type the following

```
http://{  
root * /var/www/html
reverse_proxy /api localhost:5050  
file_server
}  
```
2. Move the Caddyfile to both droplets using `rsync` command

![Creating Web App](/images/24.png)

4. In the droplets move your Caddyfile to /etc/caddy/ by running `sudo mkdir /etc/caddy` then `sudo mv Caddyfile /etc/caddy`


## Installing Node And Npm With Volta

**This is done on both your droplets**

1. In your src directory, install Volta by running `curl https://get.volta.sh | bash`
2. Run `source ~/.bashrc`
3. Run `volta install node`
4. Run `volta install npm`

## Writing A Service File

### Creating A Caddy Service File

**This is done on your local wsl machine**

1. Create and edit a service file by running `vim caddy.service` and insert the following

```
[Unit]
Description=Serve HTML in /var/www using caddy
After=network.target

[Service]
Type=notify
ExecStart=/usr/bin/caddy run --config /etc/caddy/Caddyfile
ExecReload=/usr/bin/caddy reload --config /etc/caddy/Caddyfile
TimeoutStopSec=5
KillMode=mixed

[Install]
WantedBy=multi-user.target
```

2. Transfer your caddy.service file to both your droplets using `rsync`
3. On both droplets, move caddy.service in into your system directory by running `sudo mv caddy.service /etc/systemd/system`


![Service File](/images/25.png)

4. Run these commands to restart and enable the caddy.service

```
sudo systemctl daemon-reload
sudo systemctl start caddy.service
sudo systemctl enable caddy.service
sudo systemctl status caddy.service
```


![Service File](/images/26.png)

### Creating A Node Service File

**This will be done in your local wsl host**

1. Create and edit a service file by running `vim hello_web.service` and insert the following

```
[Unit]
Description=run node application service file
After=network.target

[Service]
Type=simple
User={username}
Group={username}
ExecStart=/home/{username}/.volta/bin/node /var/www/src/index.js
Restart=on-failure
RestartSec=5
SyslogIdentifier=hello_web

[Install]
WantedBy=multi-user.target
```

2. Transfer your hello_web.service file to both your droplets using `rsync`
3. On both droplets, move hello_web.service in into your system directory by running `sudo mv hello_web.service /etc/systemd/system`


![Service File](/images/27.png)

4. Run these commands to restart and enable the hello_web.service

```
sudo systemctl daemon-reload
sudo systemctl start hello_web.service
sudo systemctl enable hello_web.service
sudo systemctl status hello_web.service
```

![Service File](/images/28.png)

## Testing and Checking Load Balancer

![Testing](/images/29.png)

![Testing](/images/30.png)

![Testing](/images/31.png)
