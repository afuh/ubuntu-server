# Ubuntu Server Initial Run

## Table of Contents
- [Access the server](#access-the-server)
  - [Generate a Key Pair](#generate-a-key-pair)
  - [Disable Password Authentication and Root login](#disable-password-authentication-and-root-login)
  - [Set Up a Basic Firewall](#set-up-a-basic-firewall)
- [Install Nginx](#install-nginx)
  - [Adjust Firewall](#adjust-firewall)
  - [Basic management commands](#basic-management-commands)
  - [Important Nginx Files and Directories](#important-nginx-files-and-directories)
  - [Add locations](#add-locations)
- [Install Node.js](#instal-nodejs)
  - [Node.js v9.x with NodeSource](#nodejs-v9x-with-nodesource)
  - [Install PM2](#install-pm2)
  - [PM2 Commands](#pm2-commands)
- [Automatic Deployment with Git](#automatic-deployment-with-git)
- [Useful commands](#useful-commands)
- [Useful links](#useful-links)


I will use `$` and `▶` to indicate the local or server terminal:
- `$`: local
- `▶`: server

## Access the server
```
$ ssh root@<server-ip>
```

Generate a new user
```
▶ adduser axel
```

As `root`, run this command to add your new user to the sudo group.
`-a` append
`-G` groups
```
▶ usermod -aG sudo axel
```

### Generate a Key Pair
[Digital ocean Keygen tutorial](https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server)
```
$ ssh-keygen
```
You will see output that looks like the following.
```
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/<localuser>/.ssh/id_rsa):
```
`/Users/<localuser>/.ssh/id_rsa` is the default path,

but you can add your own if needed `/Users/<localuser>/.ssh/id_rsa_test`

copy the key
```
$ cat ~/.ssh/id_rsa.pub | pbcopy
```

```
$ ssh root@<server-ip>
▶ su - axel
```
Create a new folder `.ssh` and restrict its permissions
```
▶ mkdir ~/.ssh
▶ chmod 700 ~/.ssh
▶ vim ~/.ssh/authorized_keys
```
Insert the key (which should be in your clipboard) by pasting it into the editor.


Restrict the permissions of the authorized_keys file with this command:
```
▶ chmod 600 ~/.ssh/authorized_keys
```
```
▶ exit
```

```
$ vim ~/.ssh/config
```

```bash
Host <login-name> #server-axel
  HostName <server-ip>
  User axel
  IdentityFile ~/.ssh/id_rsa_test #(optional)
```
If your key is located in `/Users/<localuser>/.ssh/id_rsa` and is called `id_rsa` there is no need to add the `IdentityFile` field.

Now you can enter the server with
```
$ ssh server-axel
```

### Disable Password Authentication and Root login

```
$ ssh server-axel
▶ sudo vim /etc/ssh/sshd_config
```

Change these settings
```
PasswordAuthentication no
PermitRootLogin no
```

Save the `sshd_config` file and reload the SSH daemon
```
▶ sudo systemctl reload sshd
```

### Set Up a Basic Firewall
```
▶ sudo ufw app list

Available applications:
  OpenSSH
```

We need to make sure that the firewall allows SSH connections so that we can log back in next time.
```
▶ sudo ufw allow OpenSSH
▶ sudo ufw enable
```
You can see that SSH connections are still allowed by typing:
```
▶ sudo ufw status
```

## Install Nginx
```
▶ sudo apt-get update
▶ sudo apt-get install nginx
```
### Adjust Firewall
```
▶ sudo ufw allow 'Nginx HTTP'
```
Verify the change
```
▶ sudo ufw status
```

To check if Ngnix is running type:
```
▶ systemctl status nginx
```
or just check it in Chrome with the `<server-ip>`

### Basic management commands.
- `▶ sudo systemctl stop nginx`
- `▶ sudo systemctl start nginx`
- `▶ sudo systemctl restart nginx`
- `▶ sudo systemctl reload nginx`
- `▶ sudo systemctl disable nginx`
- `▶ sudo systemctl enable nginx`

### Important Nginx Files and Directories
- `/var/www/html`: The actual web content, which by default only consists of the default Nginx page you saw earlier, is served out of the /var/www/html directory. This can be changed by altering Nginx configuration files.
- `/etc/nginx`: The nginx configuration directory. All of the Nginx configuration files reside here.
- `/etc/nginx/sites-enabled/`: The directory where enabled per-site "server blocks" are stored. Typically, these are created by linking to configuration files found in the `sites-available` directory.

### Add locations

Open the file for editing:
```
▶ sudo vim /etc/nginx/sites-enabled/default
```

```bash
server {
  listen 80;
  server_name www.axelfuhrmann.com axelfuhrmann.com;
  location / {
    root   /var/www/portfolio;
    index  index.html;
  }
  error_page  404              /404.html;
  location = /404.html {
    root   /var/www/portfolio;
  }
}

# reverse proxy
server {
  listen 80;
  server_name www.test.com test.com;
  location / {
    proxy_pass http://localhost:8080;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
  }
}
```

Make sure you didn't introduce any syntax errors by typing:
```
▶ sudo nginx -t
```

Next, restart Nginx:
```
▶ sudo systemctl restart nginx
```

## Instal Node.js
### Node.js v9.x with [NodeSource](https://github.com/nodesource/distributions)

```
▶ curl -sL https://deb.nodesource.com/setup_9.x | sudo -E bash -
▶ sudo apt-get install -y nodejs
▶ sudo apt-get install build-essential
```

### Install PM2
PM2 is a process manager for Node.js applications
```
▶ sudo npm i -g pm2
```

The `startup` subcommand generates and configures a startup script to launch PM2 and its managed processes on server boots:
```
▶ pm2 startup systemd
```
### PM2 Commands
- `▶ npm install pm2@latest -g ; pm2 update`

**Process State Management:**
- `▶ pm2 start app.js --name "api"`
- `▶ pm2 restart api/all`
- `▶ pm2 stop api/all`
- `▶ pm2 delete api/all`
- `▶ pm2 kill`

**Process Monitoring:**
- `▶ pm2 list`
- `▶ pm2 monit`
- `▶ pm2 show [app-name]`
- `▶ pm2 logs`

## Automatic Deployment with Git

```bash
▶ cd /var
▶ sudo mkdir repo && sudo mkdir repo/<site-name>.git
▶ sudo chown -R <owner>:<group> repo
▶ cd repo/<site-name>.git && git init --bare
```
`--bare` means that our folder will have no source files, just the version control.

in `/var/repo/<site-name>.git/hooks/` create the file 'post-receive' and type:
```
git --work-tree=/var/www/<site-folder> --git-dir=/var/repo/<site-name> checkout -f
```
save and set the proper permissions using:
```
chmod +x post-receive
```

In you local repository add a new remote:
```
git remote add live ssh://user@mydomain.com/var/repo/<site-name>.git
```
```
git push live master
```

## Useful commands
- `▶ sudo chown -R <owner>:<group> <folder>` Change the owner and/or group of each FILE to OWNER and/or GROUP.
- `▶ sudo tail -f /var/log/nginx/access.log` view Nginx logs
- `▶ sudo tail -f /var/log/nginx/error.log` view Nginx error logs
- `▶ ip addr show eth0 | grep inet | awk '{ print $2; }' | sed 's/\/.*$//'` check ip
- `▶ curl -4 icanhazip.com` check ip

## Useful links
- [How To Set Up a Node.js Application for Production on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-node-js-application-for-production-on-ubuntu-16-04)
- [How To Set Up Automatic Deployment with Git with a VPS](https://www.digitalocean.com/community/tutorials/how-to-set-up-automatic-deployment-with-git-with-a-vps)
