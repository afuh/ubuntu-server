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
- [Install Fail2Ban](#install-fail2ban)
  - [Changing Defaults](#changing-defaults)
  - [Configuring Fail2Ban to Monitor Nginx Logs](#configuring-fail2ban-to-monitor-nginx-logs)
  - [Adding the Filters for Additional Nginx Jails](#adding-the-filters-for-additional-nginx-jails)
  - [Activating your Nginx Jails](#activating-your-nginx-jails)
  - [Fail2Ban Commands](#fail2ban-commands)
  - [Testing the Banning Policies](#testing-the-banning-policies)
- [Using Let’s Encrypt SSL/TLS Certificates](#using-lets-encrypt-ssltls-certificates)
  - [Download the Let’s Encrypt Client](#download-the-lets-encrypt-client)
  - [Obtain the SSL certificate](#obtain-the-ssl-certificate)
  - [Automatic Renewal of Let’s Encrypt Certificates](#automatic-renewal-of-lets-encrypt-certificates)
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

To check if Nginx is running type:
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
*Follow [this guide](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-host-name-with-digitalocean) if you don't know how to set up your host name*

Open the file for editing:
```
▶ sudo vim /etc/nginx/sites-enabled/default
```

```bash
server {
  listen 80;
  server_name www.mysite.com mysite.com;
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
  server_name www.test.mysite.com test.mysite.com; # subdomains
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
- `▶ sudo npm install pm2@latest -g ; pm2 update`

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

## Install Fail2Ban
> Fail2Ban is an intrusion prevention software framework that protects computer servers from brute-force attacks.

```
▶ sudo apt-get update
▶ sudo apt-get install fail2ban
```

### Changing Defaults

We need to copy the `jail.conf` file with the contents commented out, as the basis for the our `jail.local` file. We can do this by typing:
```
▶ awk '{ printf "# "; print; }' /etc/fail2ban/jail.conf | sudo tee /etc/fail2ban/jail.local
```
```
▶ sudo vim /etc/fail2ban/jail.local
```
Important default settings:
```bash
[DEFAULT]
...
# source addresses that fail2ban ignores
ignoreip = 127.0.0.1/8

# length of time that a client will be banned. 600 seconds = or 10 minutes.
bantime = 600

# The maxretry variable sets the number of tries a client has to authenticate within a window of time defined by findtime, before being banned.
findtime = 600
maxretry = 3
```
### Configuring Fail2Ban to Monitor Nginx Logs
In the `jail.local` file there is a `JAILS` section. Each of these sections can be enabled by uncommenting the header and changing the `enabled` line to be `true`.

To enable log monitoring for Nginx login attempts, we will enable the `[nginx-http-auth]` jail.

```bash
[nginx-http-auth]

enabled = true
port    = http,https
filter  = nginx-http-auth
logpath = %(nginx_error_log)s

[nginx-botsearch]

enabled  = true
port     = http,https
filter   = nginx-botsearch
logpath  = %(nginx_error_log)s
maxretry = 2
```

This is the only Nginx-specific jail included with Ubuntu's `fail2ban` package. However, we can create our own jails to add additional functionality.

```bash
[nginx-noscript]

enabled  = true
port     = http,https
filter   = nginx-noscript
logpath  = /var/log/nginx/access.log
maxretry = 6

[nginx-badbots]

enabled  = true
port     = http,https
filter   = nginx-badbots
logpath  = /var/log/nginx/access.log
maxretry = 2

[nginx-nohome]

enabled  = true
port     = http,https
filter   = nginx-nohome
logpath  = /var/log/nginx/access.log
maxretry = 2

[nginx-noproxy]

enabled  = true
port     = http,https
filter   = nginx-noproxy
logpath  = /var/log/nginx/access.log
maxretry = 2
```
### Adding the Filters for Additional Nginx Jails
We need to create the filter files for the jails we've created. These filter files will specify the patterns to look for within the Nginx logs.
```
▶ cd /etc/fail2ban/filter.d
```
We actually want to start by adjusting the pre-supplied Nginx authentication filter to match an additional failed login log pattern.
```
▶ sudo vim nginx-http-auth.conf
```
```js
[Definition]


failregex = ^ \[error\] \d+#\d+: \*\d+ user "\S+":? (password mismatch|was not found in ".*"), client: <HOST>, server: \S+, request: "\S+ \S+ HTTP/\d+\.\d+", host: "\S+"\s*$
            ^ \[error\] \d+#\d+: \*\d+ no user/password was provided for basic authentication, client: <HOST>, server: \S+, request: "\S+ \S+ HTTP/\d+\.\d+", host: "\S+"\s*$

ignoreregex =
```
Next, we'll create a filter for our `[nginx-noscript]` jail:
```
▶ sudo vim nginx-noscript.conf
```
```js
[Definition]

failregex = ^<HOST> -.*GET.*(\.php|\.asp|\.exe|\.pl|\.cgi|\.scgi)

ignoreregex =
```

Next, create a filter for the `[nginx-nohome]` jail:
```
▶ sudo vim nginx-nohome.conf
```
```js
[Definition]

failregex = ^<HOST> -.*GET .*/~.*

ignoreregex =
```

We create the filter for the `[nginx-noproxy]` jail:
```
▶ sudo vim nginx-noproxy.conf
```
```js
[Definition]

failregex = ^<HOST> -.*GET http.*

ignoreregex =
```

And finally, we can copy the `apache-badbots.conf` file to use with Nginx.
```bash
▶ sudo cp apache-badbots.conf nginx-badbots.conf
```

To implement your configuration changes, you'll need to restart the fail2ban service. You can do that by typing:
```
▶ sudo service fail2ban restart
```

You can look at iptables to see that fail2ban has modified your firewall rules to create a framework for banning clients
```
▶ sudo iptables -S
```

### Fail2Ban commands

- `▶ sudo fail2ban-client status` see the enable jails
- `▶ sudo fail2ban-client status nginx-http-auth`: see the status of that particular jail

### Testing the Banning Policies
From another server, we can test the rules by getting our second server banned. After logging into your second server, try to SSH into the fail2ban server. You can try to connect using a non-existent name for instance:
```bash
# second sever
▶ ssh blah@fail2ban_server_IP
```
Repeat this a few times. At some point, the fail2ban server will stop responding.

On your fail2ban server, you can see the new rule by checking our iptables again:
```
▶ sudo iptables -S
```
```bash
...
-A INPUT -j DROP
-A fail2ban-nginx-http-auth -j RETURN
-A fail2ban-ssh -s 203.0.113.14/32 -j REJECT --reject-with icmp-port-unreachable # fail2ban Jail
-A fail2ban-ssh -j RETURN
```

## Using Let’s Encrypt SSL/TLS Certificates
Let’s Encrypt is a certificate authority (CA) offering free and automated SSL/TLS certificates. Certificates issued by Let’s Encrypt are trusted by most browsers in production today.

### Download the Let’s Encrypt Client
Install certbot
```
▶ add-apt-repository ppa:certbot/certbot
▶ sudo apt-get update
▶ sudo apt-get install python-certbot-nginx
```
### Obtain the SSL certificate
Certbot has various plugins to generate SSL certificates. The NGINX Plugin will take care of re-configuring NGINX and reloading the configuration whenever necessary.

To generate SSL certificates with the NGINX plugin, run the following command:
```
▶ sudo certbot --nginx -d example.com -d www.example.com
```
Once the process has completed successfully, certbot will prompt you to configure your HTTPS settings, which includes entering your email address and agreeing to the Let’s Encrypt terms of service.

We need to make sure that the firewall allows HTTPS connections.
```
▶ sudo ufw allow 'Nginx HTTPS’
```

### Automatic Renewal of Let’s Encrypt Certificates
Let’s Encrypt certificates expire in 90 days. We encourage you to automatically renew your certificates when they expire. We’ll set up a cron job to do this.
```
crontab -e
```
we enter the certbot command we wish to run daily. In this blog post, we run the command every day at noon. The command will check to see if the certificate on the server will expire within the next 30 days, and renew it if so.

```bash
0 12 * * * /usr/bin/certbot renew --quiet
```

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
- [How To Set Up a Host Name with DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-host-name-with-digitalocean)
- [How To Set Up a Node.js Application for Production on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-node-js-application-for-production-on-ubuntu-16-04)
- [How To Set Up Automatic Deployment with Git with a VPS](https://www.digitalocean.com/community/tutorials/how-to-set-up-automatic-deployment-with-git-with-a-vps)
- [How To Set Up a Host Name with DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-host-name-with-digitalocean)
- [How To Protect SSH with Fail2Ban on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-14-04)
- [Using Free Let’s Encrypt SSL/TLS Certificates with NGINX](https://www.nginx.com/blog/using-free-ssltls-certificates-from-lets-encrypt-with-nginx/)
