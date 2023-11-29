This tutorial is made under the assumption that the reader has already created a Debian 12 droplet on Digital Ocean and has already linked their personal .ssh key to the droplet

# 1. Login to your droplet using your SSH key as root user

```bash
ssh -i path-to-your-key root@<ip-address of your droplet>
```
- Since this is the first time you are logging into the droplet, you will need to agree to connecting to the server. It will kick you out after adding the server to your list of known hosts.
- simply put the command in again to get into the server

For example:
```bash
ssh -i .\.ssh\do-key root@143.198.154.184
```
# 2. Create a new regular user
 - You are now logged in as the "root" user. However, for security purposes, it is important to create a regular user instead.
 - The command to run is as follows:

```bash
useradd -ms /bin/bash <user-name>
```

Here's a breakdown of the command:
 - useradd: This is the linux command to create a new user
 - -m: This is an option for useradd to create a home directory for the new user
 - -s /bin/bash: This specifies the login shell for the new user. In this case, it is the Bash shell
 - Replace the user-name with one of your choice

# 3. Allow new user to do administrative tasks
 - Now that the user has been created, we need to give the user administrative privileges
 - The command to run is as follows:
```bash
usermod -aG sudo <user-name>
```

Here's a breakdown of the command:
 - usermod: Linux command to modify user accounts
 - -aG: **A**ppend  the user to a **g**roup of your choosing. In this case, append the user to the "sudo" group, which is responsible to execute any command as the root user if it is prefaced with sudo

 - You should also provide the user with a password in order to use sudo commands
```bash
sudo passwd <user-name>
```
 - The system will prompt you to enter a password of your choosing
# 4. Allow the new user to connect to the droplet via SSH
 - We want the user to be able to log in to the droplet directly. There are a few things to prepare before we are able to do so.
 1. Copy the .ssh directory from the root user to the new user
```bash
cp -r /root/.ssh /home/<user-name>
```
Here's a breakdown of the command:
 - cp: Linux command to copy files or directories
 - -r: An option when copying a directory to do so recursively (copy the directory **as well as** it's contents)
 - /root/.ssh: The source directory we are copying
 - /home/user-name: The destination directory

2. Change to the new user's home directory
```bash
cd /home/<user-name>
```
 - You should have a .ssh file in this directory. You can check by running the following command:
```bash
ls -a
```
 - Since the .ssh file is hidden, we use the "-a" option

3. Change the ownership of the .ssh file so that the user and group are under the new user
 - We want the user to be able to log in using the ssh file, but they cannot do this if they do not own the .ssh file as well. To change this, run the following command:
```bash
chown -R <user-name>:<group-name> /home/test/.ssh
```
Here's a breakdown of this command:
 - chown: Linux command to **ch**ange **own**ership of a file or directory
 - -R: Similarly to cp -r, chown -R recursively changes the directory owner. It is important to note that you **MUST*** use "-R" for chown, as it is not interchangeable with "-r"
 - user-name:groupname : These are the target owner and group of the directory. In this case, replace both user-name and group-name with your user-name.
 - /home/test/.ssh: The target file that we are changing ownership of

4. Test the login method
- To ensure that your user is correctly set up, logout of the droplet, and attempt logging in with your new user
- To do so, replace the "root" of your original login command with your username
```bash
ssh -i path-to-your-key user-namet@<ip-address of your droplet>
```

# 5. Prevent the root user from connecting to the server via SSH
 - Now that we have created an administrative user, we want to prevent anyone logging in as the root user. We do this because the root user has unlimited privileges to execute any command. It can be used maliciously to harm the system. However, even non-maliciously, mistakes made in the root user can cause critical errors. Provided the root user is accessible by multiple individuals, it can be difficult to determine which user changed something when logged in.
 - In order to change the ssh file, we need to locate the `sshd_config` file, which is located in /etc/ssh
```bash
cd /etc/ssh
```

 - To edit the `sshd_config` file, we use a command called "vim", which is a text editor. We will need to use the admin privileges to change any file in the /etc folder by prefacing our commands with "sudo". The system will prompt you for the password you made earlier
```bash
sudo vim sshd_config
```

 - Once you are in the text editor, press "I" to go into "Insert" mode
 - Look for the line that says `PermitRootLogin` and change its value to "no"
 - Press escape, then ":wq" to **w**rite and **q**uit the file

 - Once complete, we need to restart the SSH service since we changed the `sshd_config` file
 - Run the following command:
```bash
systemctl restart ssh.service
```
Here's a breakdown of the code:
 - systemctl: A Linux utility that allows you to control the systemd system and service manager. In this case, we are managing daemons
 - restart: Indicate to the system to stop and start a specified service
 - ssh.service: This file handles all incoming SSH connections, and thus, parses the sshd_config file

# 6. Install nginx
 - Nginx is a open-source web server and reverse proxy server, allowing it to provides static files and/or control and direct traffic.
 - To install, we run the following:
```bash
sudo apt install nginx
```
Here's a breakdown of the code:
 - apt: This is a package handling tool in Linux
 - install: Using the apt command, install a new package

# 7. Configure nginx to serve a sample website
 - Now that we have installed nginx, we want to serve a website

1. Documents that are served by nginx are stored in the /var/www directory by default, so we will navigate there:
```bash
cd /var/www
```

2. Since you may wish to serve multiple websites on the same server, it will be important to create a directory to save all related files together for organization's sake. For this example, we will call the directory "my-site". You will need the sudo command.
```bash
sudo mkdir my-site
```
 - mkdir is the linux command to make a directory
 - Change into the newly made directory with "cd"
```bash
cd my-site
```

3. Create a simple webpage
 - Create an HTML file for the main page of on the site
```bash
sudo vim index.html
```
 - For this example, we will be using the following code in the index.html:
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>2420</title>
    <style>
        body {
            display: flex;
            align-items: center;
            justify-content: center;
            height: 100vh;
            margin: 0;
        }
        h1 {
            text-align: center;
        }
    </style>
</head>
<body>
    <h1>Hello, World</h1>
</body>
</html>
```

4. Remove default symbolic links
- Nginx will have symbolic links to a default file.
- The symbolic links are present in the `/etc/nginx/sites-enabled` and `/etc/nginx/sites-available` directories. We will need to change into those directories and unlink them with the following commands:
```bash
cd /etc/nginx/sites-enabled
sudo unlink default

cd /etc/nginx/sites-available
sudo unlink default
```

5. Create a server block
 - In the `/etc/nginx/sites-available` directory, we need to create a nginx config file to serve our index.html
 - First, create the my-site.conf file:
```bash
sudo vim my-site.conf
```
 - For this example, put the following code into the file:
```
server {
	
	listen 80;
	listen [::]:80 default_server;

	#decide where to look for main files
	root /var/www/my-site;
	
	#Look for index.html
	index index.html;

	#Domain name
	server_name _;

	#Routing
	location / {
		# First attempt to serve request as file, then
		# as directory, then fall back to displaying a 404.
		try_files $uri $uri/ =404;
	}
}
```

6. Create a symbolic link between your new config file to `/etc/nginx/sites-enabled`
 - It is common practice to create symbolic links between the `sites-available` directory and the `sites-enabled` directory, as it allows all the server block configurations to be kept in one place (`sites-available`), and choose which ones to enable without having to move or duplicate configuration files.
 - You can do so with the following command:
```bash
 sudo ln -s /etc/nginx/sites-available/my-site.conf /etc/nginx/sites-enabled
```
Here's a breakdown of this code:
 - ln: command to create a link between two files/directories
 - -s: ln option to create a symbolic link instead of a hard link

7. Refresh nginx to reload the data
 - Now that we have a new symbolic link, we need to refresh the page so it updates to index.html instead of default
 - Use the following command:
 ```bash
 sudo systemctl restart nginx
```

 - Check and make sure your nginx is running properly using the following command:
```bash
sudo nginx -t
```

8. Display the page
 - Provided there are no errors, you should be able to run the page. There are two ways to check.
 - You can run the below command, where the ip-address is the ip of your server:
```bash
curl <ip-address>
```
 - Or, you can directly type the ip-address into your browser. Both should provide you with similar results.