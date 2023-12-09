# Assignment 3 Part 1

Assuming the reader has already set up a Debian 12 Server on a droplet in DO;

This tutorial will cover the following steps:

1. Creating a new regular user that can:
    - Preform Administrative tasks ( have sudo access )
    - Has bash as the login shell
    - Can access the server via SSH
2. Prevent the root user from connecting via SSH
3. Install nginx
4. Configure nginx to serve a sample website

## Creating a New Regular User

After logging in to the Debian 12 server as root, use the `useradd` command to create the new user. We'll name the user leon1.

```
useradd -m leon1
```

To allow the new regular user (leon1) to preform administrative tasks, we need to give the user sudo permissions.

We will do this by adding the user to the `sudo` group

```
usermod -aG sudo leon1
```

---
On Debian, `sh` is typically set as the default login shell for new users. To check if it is, look for the user (in this case leon1) inside `/etc/passwd` like this:

```
grep leon1 /etc/passwd
```

At the end of the resulting code, it might look like this: 

`.../home/leon1:/bin/sh`

indicating that the login shell is sh. 

We want to change the login shell to bash, so we're going to use `chsh`

```
chsh -s /bin/bash leon1
```

And the login shell will be changed to bash.

---
To access the server via SSH, we need to first create an SSH directory in the user's home directory.

```
mkdir -p /home/leon1/.ssh
```

We then need to copy the public key (from root) to the new user's `.ssh` directory we just created.

```
cp /root/.ssh/authorized_keys /home/leon1/.ssh/
```

We'll then need to set the correct permissions of the `.ssh` directory to the new user.

```
chown -R leon1:leon1 /home/leon1/.ssh
```

After that step, you should be able to connect to the server using 
```
ssh -i /path/to/private-key leon1@server-ip
```

Congratulations! You have created a new user with administrator permission, the login shell as bash, and gave it access to the server via SSH.

## Preventing the root user from connecting to the server via SSH

Preventing the root user from logging in via SSH is good security practice. It prevents just anyone from gaining full control over the entire system.

We'll do this by editing the SSH Configuration file using vim.

```
vim /etc/ssh/sshd_config
```

Find the line that says `PermitRootLogin yes` and change it to `PermitRootLogin no`. To do this, enter INSERT mode by pressing `i` on your keyboard

If the line is commented out with `#` before it, uncomment it by removing the `#`

Save the file by pressing ESC on your keyboard, and then entering `:wq`

We need to restart the SSH service to apply the changes. 

```
systemctl restart sshd
```

Congratulations! You have prevented the root user from connecting to the server via SSH.

## Installing Nginx

Nginx is a web server that can be used to server a personal website. We'll download that by using `apt`.

```
apt install nginx
``````
## Configuring Nginx to Serve a Sample Webiste

We need to configure a directory for the Website. They are stored in `/var/www`, so we'll need to add a new directory with the name as our desired domain. Let's just use `domain1` as the domain name for this example.

```
mkdir -p /var/www/domain1
```

We need to change the owner of the directory to the user we just created (`leon1`). We'll do so using `chown`:

```
chown -R leon1:leon1 /var/www/domain1
```

In this directory, we need to create the base html for the website. We will use `vim` to create an `index.html` file.

```
vim /var/www/domain1/index.html
```

Following the previous instructions to access vim INSERT mode, type in a block of HTML code that will serve as the sample website. It could look like this:

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>leon's website</title>
</head>
<body>
    <h1>hello! this website is working. </h1>
</body>
</html>
```

Save the file using `:wq` again.

We then need to configure the server block by creating a configuration file in Nginx's `sites-avaliable` directory.

```
vim /etc/nginx/sites-available/domain1.conf
```

Add the following into the configuration file.

```
server {
    listen 80;
    listen [::]:80;

    root /var/www/domain1;
    index index.html index.htm index.nginx-debian.html;

    server_name _;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

To enable this file, we need to create a symbolic link to the `sites-enabled` directory using `ln`.

```
ln -s /etc/nginx/sites-available/domain1.conf /etc/nginx/sites-enabled/
```

We'll want to test the Nginx configuration for any errors by running:

```
nginx -t
```

After nginx confirms that syntax is ok and the tests is successful (everything is running and working properly), we will restart Nginx to apply the changes.

```
systemctl restart nginx
```

Congratulations! You have successfully Installed Nginx and configured it to serve a sample website.

