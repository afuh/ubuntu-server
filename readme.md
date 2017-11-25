# Ubuntu server first run
[How To Set Up a Node.js Application for Production on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-node-js-application-for-production-on-ubuntu-16-04
  )

- `$` local
- `▶` server


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
Host <login-name>
  HostName <server-ip>
  User axel
  IdentityFile ~/.ssh/id_rsa_test #(optional)
```
If your key is located in `/Users/<localuser>/.ssh/id_rsa` and is called `id_rsa` there is no need to add the `IdentityFile` field.

Now you can enter the server with
```
$ ssh <login-name>
```
