

**1.Initial Setup**
-   Log in to Microsoft Azure using an existing Microsoft account, or create one. Search ‘Azure for students’ and you will be asked for your college email id. Using that you will avail 100 free credits. You can use that to create your first VM.
-   Create a new resource, and select virtual machine. Fill in details such as vm name, your vm image(ubuntu 20.04 in my case) etc. Now for size, select the standard B1s -1vcpu,1 GiB, as it’s the least priced and free services eligible.
-   Now, in the next step, you would have to select the authentication type you want to choose, I recommend selecting a SSH public key, as its more secure than a password.
![](https://i.ibb.co/cTNQLDL/Screenshot-from-2024-06-25-00-07-07.png)
-   Once your vm has been created, you will be assigned a local ip address. Note it down.
-   Open your terminal and type the command – `ssh [username@ipaddress]`, afterwards enter the password, and enter your vm
-   If you instead opted for ssh key pair, then the private key will be downloaded in your system and the public key will be already present in the .ssh file of your vm user. Use `ssh -i /path_to_private.key [username@ipaddress]` to log in.
-   You can maybe create a config file in the .ssh directory of your system to automate things,
    do `sudo nano .ssh/config` and edit as follow- after which you can log in to your vm using `ssh name`
    ![Adding a host to your config file](https://i.ibb.co/tKxZc1w/Screenshot-from-2024-06-25-00-17-29.png)
 -   Make sure to update the system packages after accessing your VM. Use the following commands -
	    -   `sudo apt update`
	    -   `sudo apt upgrade`
-   Now, for unattended-upgrades, install it using `sudo apt install unattended-upgrades`. Now use a text editor of your choice to edit the file `‘/etc/apt/apt.conf.d/50unattended-upgrades’` and uncomment the lines given below.
	  	 
	![enter image description here](https://i.ibb.co/jfhbvNR/Screenshot-from-2024-06-25-00-24-39.png)
-   Next open the 20auto-upgrades, and add the following lines(Here ‘7’ means it will automatically upgrade every seven days, you can set your own value)-
-![enter image description here](https://i.ibb.co/Vpk6zFb/Screenshot-from-2024-06-25-00-26-54.png)

Just run `‘sudo systemctl status restart unattended-upgrades.service’` to update the new configurations.

**2.Enhanced SSH Security**

 -   To achieve things like disabling root login and password authentication you have to open the `/etc/ssh/sshd_config` file and change the following lines.
![enter image description here](https://i.ibb.co/kH7sjx7/Screenshot-from-2024-06-25-00-50-35.png)
![enter image description here](https://i.ibb.co/wW8PCMj/Screenshot-from-2024-06-25-00-51-00.png)
  

 -   For PublicKey Authentication the default value will be on, now for generating a key pair run the command(not on VM,but on system)-
    
    -   `ssh-keygen -t rsa -b 2048`(generates a key-pair of rsa form)
        
    -   `ssh-copy-id username@ipaddress`
        
    -   save the private key in your .ssh folder and copy the public key in to authorized_keys of your server’s .ssh directory.
        
 -   Restart ssh using the command `sudo systemctl restart ssh`
    
 -   Now for setting up fail2ban, install it using the commands-
    
    -   `sudo apt update`
        
    -   `sudo apt install fail2ban -y`
        
 -   Now make a copy of the jail.conf named jail.local located in the /etc/fail2ban directory using-
    
    -   `sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local`
        
 -   Open the jail.local using a text editor-
    
    -   You will be able to see some sections with some defined rules.
        
    -   You need to find the sshd section and add these configs-
    -![enter image description here](https://i.ibb.co/g4K5f6W/Screenshot-from-2024-06-25-01-53-38.png)
    -   Restart the fail2ban service using `sudo systemctl restart fail2ban`
    
 -   To give access to the remote user, copy the contents of the publickey provided and add it to the authorized_keys file in the .ssh directory of a user with sudo privileges.

**GOOGLE AUTHENTICATOR:**
 -   Install it using – `sudo apt-get install libpam-google-authenticator`
 -   Open the `/etc/pam.d/sshd` using a editor and add the following line at the end--![enter image description here](https://i.ibb.co/HCDfBJy/Screenshot-from-2024-06-25-02-15-10.png)
 -   Now we have to configure SSH also to allow MFA, open the `/etc/ssh/sshd_config` using an editor and change the following configs-![enter image description here](https://i.ibb.co/93t20Zs/Screenshot-from-2024-06-25-02-18-28.png)![enter image description here](https://i.ibb.co/BcLFmm1/Screenshot-from-2024-06-25-02-19-12.png)
 - Now restart ssh using `sudo systemctl restart ssh`
 - Now run the command `google-authenticator` and scan the qr code on your authenticator app, to set up MFA.

**3.Firewall and Network Security:**

 -   Now to change the default port(22), first go to your Azure platform and select the nsg(network security group) associated with your VM, then go to Inbound security rules and add the non-default port, https(443) and https(80) rules.
	![enter image description here](https://i.ibb.co/YtpqJd2/Screenshot-from-2024-06-28-23-53-01.png)
 -   Next, start your server and then open the `/etc/ssh/sshd_config` and change this line(according  to the port you are selecting)-
 ![enter image description here](https://i.ibb.co/sCScBPK/Screenshot-from-2024-06-25-09-17-59.png)
 -   Install UFW(Uncomplicated firewall) using the commands- 
	    -   `sudo apt-get update`
	    -   `sudo apt-get install ufw`
 -   Now by default ufw will deny all ports, so to allow aceess to these specific ports, do-
    -   `sudo ufw allow 80/tcp`
    -   `sudo ufw allow 443/tcp`
    -   `sudo ufw allow Non-default Port No/tcp`
 -   Then just enable ufw using the command `sudo ufw enable`.
 -   Check the status of the ports allowed using `sudo ufw status`, which will show a table of allowed ports.
![enter image description here](https://i.ibb.co/rsN2xbn/Screenshot-from-2024-06-25-09-27-29.png)
 -   To enable ufw logs, use the command `sudo ufw logging on`. The logs will be found in either `/var/log/ufw.log` or `/var/log/syslog`.


**4. User and Permission Management:**

 -   To create the list of users given, use the command-
    -   `sudo adduser ‘username’`
    -   Use the usernames like exam_1,examadmin,examaudit.      
    -   Also set passwords for each using `sudo passwd ‘username’`        
 -   Now to give the user examadmin root we have to add him to the sudo group, which can be done through-
    -   `sudo usermod -aG sudo examadmin`
    -   This command adds the user examadmin in the group sudo, giving them root privileges.
 -   Now to ensure that examaudit has read-only access to the exam users, we need the ACL package(Access Control Lists).
    -   Install it using -
    -   `sudo apt-get update` and `sudo apt-get install acl`
    -   Then give read-access to examaudit using the command-
    -   `sudo setfacl -m u:examaudit:r /home/exam_1`
    -   Repeat this for all exam users
    -   You can check your acl settings using `getfacl /home/’username’`
 -   Now to ensure that each user’s home directory is only accessible by that user only, we need to run the command -
    -   `sudo chmod 700 /home/username`
    -   Run this command for every user you have.
    -   Verify this by running the commmand `ls -l`. The image below shows that the directory is only accessible for read,write and execute for the owner, and none for the other users in same groups or other users.
    ![enter image description here](https://i.ibb.co/K2xpDqG/Screenshot-from-2024-06-26-00-45-35.png)
    -   The plus in front of the exam_users also suggest that they are read-accessible by the examaudit.
    
 -   Now create a backup script according to what you want to name it, probably in `/usr/local/bin` directory.
 -   To make sure that only examadmin can run the script,
    -   `sudo chown examadmin:examadmin /usr/local/bin/backup.sh`, so examadmin becomes the owner of the script.
    -   `sudo chmod 700 /usr/local/bin/backup.sh` ,so only the owner of the file read,write and execute permissions of the script.
    -   Also run `sudo chmod +x /usr/local/bin/backup.sh`, to make the script an executable file.
 -   Now in your script
    -   Add `#!/bin/bash` at the top to let the system knows it’s a bash script/
    -   Create a variable which points to your backup_directory.
    -   Iterate through the exam users and `tar -czf backup_dir/user_home.tar.gz /home/user` to make a compressed tar file of the home directory of exam users.
    -   Also add a date configuration as it would be run daily.
    -   Also make sure that the backup directory is already created, and its permissions and owners are set accordingly. If not run the commands `sudo chmod 700 backup_dir` and `sudo chown examadmin:examadmin backup_dir`.
 -   Now to automate the script, use crontab as the examadmin using-
    -   `sudo crontab -u examadmin -e` to open the crontab and select the editor you want to use.
    -   Now add a line at the end according to the time you want to run the file.   ![enter image description here](https://i.ibb.co/bdtwqK7/Screenshot-from-2024-06-26-01-09-36.png)
    -   Here the syntax is -
	    -   m -minute
	    -   h -hour
	    -   dom -day of month
	    -   mon -month
	    -   dow-day of week    
	-   Here our crontab ensures that the script runs at 2:00 am daily.


**5. Web Server Deployment and Secure Configuration:**

 - To install nginx, run the following commands -
	 
	 - `sudo apt update`
	 - `sudo apt install nginx`
	 - `sudo systemctl enable nginx`, to start nginx on boot.
	 - `sudo systemctl start nginx` 

 -  Now create a simple user as told in the above steps.
 - The next step is to download both the apps.
 - Note: Make sure you go into the home directory of the new user you created.
 - For APP1:
	 
	 - Download it using the command,  `wget https://do.edvinbasil.com/ssl/app`
	 - Next, download the sha256 signature using, `wget https://do.edvinbasil.com/ssl/app.sha256.sig`
	 - Just verify the sha256 checksum using, `sha256sum -c app1.sha256.sig`, it would give output as app1:OK on your terminal if its been successfully downloaded.
	 - Next to make it executable, `chmod +x app1`
	 - Run the app using `./app1` and it will show this on your terminal.
	 ![enter image description here](https://i.ibb.co/B3MsJ5g/Screenshot-from-2024-07-10-18-12-21.png)

 - For APP2:
	 
	 - First clone the repository using `git clone https://gitlab.com/tellmeY/issslopen`
	 - Even though it's said that we can use either bun or docker, but I recommend using docker as I found it more convenient. So, make sure to install docker and docker-compose.
	 - Next, make sure you are in the issslopen directory, and make a copy of the .env.example file using `cp .env.example .env`
	 - Next make sure to edit the ADMINS variable and add two tokens for users.
		 ![enter image description here](https://i.ibb.co/HpwLPch/Screenshot-from-2024-07-10-18-58-36.png)

	 - Next run the application using `docker-compose up -d`
	 - You can update the image use `docker-compose pull`
	

 - Next the task was to setup a reverse proxy server using Nginx for our domain, but I failed despite numerous attempts still I would try to explain the major process I did for it.
	 
	 - First make a folder named sites-available in your /etc/nginx directory and then in that directory make a file sharing the same name as your domain.
	 - Then edit the configurations of that file using a text editor.
		 - ![enter image description here](https://i.ibb.co/nCXgbLJ/Screenshot-from-2024-07-10-19-47-28.png)

	 - Ok, also generate a ssl certificate suing certbot, for that follow these steps-
		 
		 -  Install certbot using `sudo apt update && sudo apt install certbot python3-certbot-nginx`
		 - Then obtain a certificate using `sudo certbot certonly --webroot -w /var/www/html -d domain_name`
		 - Make sure to mention the correct paths of your certificate and certificate key as shown in the above file.
		 - I was not able to obtain a proper certificate, I still haven't figured out the issue.
		 - Next, use this command to check if your configuration has any errors `sudo nginx -t`
		 - Reload the Nginx using `sudo systemctl reload nginx`, and visit your domain.


**6. Database Security:**

 - To install MariaDB, follow the given commands-

	 - `sudo apt update`
	 - `sudo apt install mariadb-server`
	 - `sudo mysql_secure_installation`
	 - The above command will enable you to adjust some settings like disallowing root login remotely, removing test database, remove anonymous users and set a root password.

 - Now when you have to use the MariaDB shell, you have to enter as the root user using the command `sudo mysql -u root -p` ,and enter the password you set.
 - Once in the MariaDB shell, create the database using the MySQL command `CREATE DATABASE secure_onboarding`.
 ![enter image description here](https://i.ibb.co/gMzFZt3/Screenshot-from-2024-06-29-05-06-27.png)
 - Now, to create a user with minimal privileges.

	 - `CREATE USER 'username'@'localhost' IDENTIFIED BY 'password';`
	 - This will just create the user, now to grant him minimal privileges.
	 - `GRANT SELECT, INSERT, UPDATE, DELETE ON secure_onboarding.* TO 'username'@'localhost';`
	 - `FLUSH PRIVILEGES;` , this updates the privileges of the user immediately.
	 - Then exit the MariaDB shell.

 - You can disable remote root login during mysql_secure_installation, but in case you missed it, enter the MariaDB Shell, and enter the following commands-
	 
	 - `DELETE FROM mysql.user WHERE User='root' AND Host NOT IN ('localhost');`
	 - `FLUSH PRIVILEGES;` ,to update the new settings.
	 - To verify this, we can check it using `SELECT User, Host FROM mysql.user WHERE User='root';`	![enter image description here](https://i.ibb.co/C8nGXtV/Screenshot-from-2024-06-29-05-24-21.png)
		 

 - Now to ensure MariaDB is only accessible from only our localhost, we have to do the following steps-
	 

	 - Go to the `/etc/mysql/mariadb.conf.d` and open the `50-server.cnf` using a text editor of your choice.
	 - Find the `[mysqld]` section and add the following line-	 ![enter image description here](https://i.ibb.co/30wL97X/Screenshot-from-2024-06-29-05-29-58.png)
	 - Now exit the editor and restart MariaDB, using the command `sudo systemctl restart mariadb` ,to update the new settings.

 

 - To create a backup script-
	 
	 ![enter image description here](https://i.ibb.co/mhPd3SM/Screenshot-from-2024-06-29-05-43-22.png) 
	 - Create a directory to store your backups, and store it in a variable. Assign variables for the user, password and database too.
	 - Now also set a name for your backup file in your .sql format, preferably the directory name along with the date.
	 - Use the `mysqldump` command shown in the script, which would transfer the contents of the database to the backup file that we specified.
	 - Also make this script executable using the command `sudo chmod +x 'path to script'`
	 - Next to automate the backup process, open a Crontab with `crontab -e` and add a command like we used for the exam user backup script.
![enter image description here](https://i.ibb.co/MkKTGmh/Screenshot-from-2024-06-29-05-51-59.png)
This ensures that the script runs daily at 12:00 am.

**7.VPN Configuration:**

 - To install wireguard, run these commands-

	 - `sudo apt update`
	 - `sudo apt install wireguard`

 - Now, generate a public-private key pair preferably in the `/etc/wireguard` directory where you will be writing your configuration file. Use the following command for this.
	 
	 - `wg genkey | tee /etc/wireguard/server_private.key | wg pubkey > /etc/wireguard/server_public.key`
	 - This line generates a key pair, and stores the private key and public key in respective files.

 - The next step is to make a configuration file for your VPN, go to the `/etc/wireguard` and create a file and name it accordingly( usual conventions follow wg(1,2,3), vpn(0,1,2) etc.). I decided to go with wg0.conf .
	 
	 -   Open the wg0.conf file with a text editor and add the following configurations.	 ![enter image description here](https://i.ibb.co/VgvmMCS/Screenshot-from-2024-06-29-15-07-17.png)
	 - This sets up the interface portion from your side. 
	 - Note: Most of the times people use 51820 as the listen port, but make sure to allow ufw for whichever port you use, using the command `sudo ufw allow 51820/other port no.`
	 - The conf file should have root ownership, and can only be read or written by the owner. Set this using the commands
	 - `sudo chown root:root /etc/wireguard/wg0.conf`   and   `sudo chmod 600 /etc/wireguard/wg0.conf.`

 - We also need to enable ip forwarding open the file `/etc/sysctl.conf` using an editor and add the following line-
![enter image description here](https://i.ibb.co/wydhvmM/Screenshot-from-2024-06-29-15-16-45.png)
	

	 - Use the command `sudo sysctl -p` , to update the changes.
	 - This will ensure that the VPN allows access to the local network and the internet.

 - Now, we have to similarly generate 2 key pairs for our clients and for that we will use the the previous command-  `wg genkey | tee /etc/wireguard/client(1/2)_private.key | wg pubkey > /etc/wireguard/client(1/2)_public.key`
 - Next we have to add the configurations for the clients in our wg0.conf file in the `/etc/wireguard` directory, shown as below-
![enter image description here](https://i.ibb.co/pw0wsf0/Screenshot-from-2024-06-29-15-07-38.png)

	 - Don't forget to share the respective private keys to your client so they can access the VPN.
 - NOTE: Don't forget to set proper permissions for your key pairs using the following commands.
	
	- `sudo chown root:root 'path to all keys'` , setting root ownership for the keys.
	- `sudo chmod 600 'path to all private keys'`, setting owner only read and write permission for owner.
	- `sudo chmod 644 'path to all public keys'`, setting owner only write, but read for all users.
 - Now enable the wireguard interface on your server, using the following commands- 
	
	- `sudo systemctl enable wg-quick@wg0`
	- `sudo systemctl start wg-quick@wg0`  

 - You can bring up and bring down the interface using the command-
	 
	 - `sudo wg-quick up wg0`
	 - `sudo wg-quick down wg0`

 - To check the current status of the interface we can use the command-
	 
	 - `sudo wg show`, this command will show the status of all active interfaces.
	 - For example, if we consider the interface wg0 that we set up, it will show this-
	 ![enter image description here](https://i.ibb.co/x5H9vBw/Screenshot-from-2024-06-29-15-54-23.png)
	 

 - If in any case, the peers of your interface are not showing up, then you can manually add them using the commands-
	 
	 -  `sudo wg set wg0 peer 'Client 1 Public Key' allowed-ips 10.0.0.2/32`
	 - `sudo wg set wg0 peer 'Client 2 Public Key' allowed-ips 10.0.0.3/32`
	 - Again just verify it using `sudo wg show`.

	 

  

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTQ0MTU0MTI0OV19
-->