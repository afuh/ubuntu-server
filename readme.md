# Ubuntu server first run
[How To Set Up a Node.js Application for Production on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-node-js-application-for-production-on-ubuntu-16-04
  )

- `$` local
- `▶` server

## Initial setup
### Access the server
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

## Disable Password Authentication and Root login

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

## install Nginx
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
