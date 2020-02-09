# Install Apache2-Proxied HTTPS JupyterHub Service Running as Root or with Sudo on Ubuntu 18.04

This how-to is based on the installation instructions available from Jupyterhub, [Install JupyterHub and JupyterLab from the ground up](https://jupyterhub.readthedocs.io/en/latest/installation-guide-hard.html), also known as "the hard way."

It also utilizes an article illustrating how to [run JupyterHub as a non-root user](https://github.com/jupyterhub/jupyterhub/wiki/Using-sudo-to-run-JupyterHub-without-root-privileges).

Note that these instructions do not adequately address additional installation and use issues. For example, it uses PAM for authentication of users with accounts on the same machine as the server.

Also, if copying and pasting from this document, note that the two examples have different installation paths. 

### Preparation

Ubuntu 18.04 server edition includes these essentials.

	$ sudo apt install openssh-server curl git software-properties-common

Install Apache2 and Certbot

	$ sudo apt install -y apache2

	$ sudo add-apt-repository ppa:certbot/certbot

	$ sudo apt update && apt install -y python3-certbot-apache


Install NodeJS and NPM. (Ubuntu repo has old versions, but they are sufficent here)

	$ sudo apt install -y nodejs npm

Install configurable-http-proxy for Node, made by JupyterHub

	$ sudo npm install -g configurable-http-proxy

### Install Using Anaconda and Run as Non-Root

I recommend this approach. I previously thought it was overkill to install so many Python packages; but it is tidy and includes so much of what you might want to use anyway. 

Anaconda is optional, but running the server publicly as root is not.


##### JupyterHub Users

Create a group for jupyterhub users and add the users

	$ sudo addgroup jupyterhub

Create a user to control Jupyterhub

	$ sudo adduser hubber

Make it a member of the jupyterhub group. Additional new users can be added in the same way.

	$ sudo usermod -a -G jupyterhub hubber

	$ sudo usermod -a -G jupyterhub me

Jupyterhub users will have shell access, so make everyone's home directory inaccessibe to other users. 

	$ sudo chmod o-rwx /home/*


##### Anaconda

Create a directory for anaconda, with hubber as the owner

	$ sudo mkdir /opt/hubber

	$ sudo chown hubber:hubber /opt/hubber


Get and run the installation script. 

    Note that the version is "2019.10" does not refer to the Ubuntu version and will work just fine. 

    Also, do not use "sudo" even though their instructions suggest it. 

    Do use "bash" even if you are already in a bash shell. 

    Script will prompt for location; I entered "/opt/hubber/anaconda". 

    I also selected automatic activation and to "conda init" at end of process.

	$ wget https://repo.anaconda.com/archive/Anaconda3-2019.10-Linux-x86_64.sh

	$ bash Anaconda3-2019.10-Linux-x86_64.sh


The installation modifies the user's .bashrc, including adding the virtual-environment indicator, "(base)", to the shell prompt. For customized Bash prompts, reference $CONDA_PROMPT_MODIFIER in PS1 to indicate the active environment.


##### SudoSpawner

Install the "sudospawner" program from jupyterhub

	(base) $ pip install sudospawner

Configure sudo to allow hubber to use sudospawner on behalf of other jupyterhub users.

Create a sudo configuration file in /etc/sudoers.d/jupyterhub-sudo-conf. (see README in that directory for information)

	$ sudo nano /etc/sudoers.d/jupyterhub-sudo-conf

Copy and paste therein these lines:                      

	# an alias allows password-free usage  

	Cmnd_Alias SPAWNER_CMD = /opt/hubber/anaconda/bin/sudospawner

	# allow any jupyterhub member to spawn single-server as hubber

	%jupyterhub ALL=(hubber) /usr/bin/sudo

	hubber ALL=(%jupyterhub) NOPASSWD:SPAWNER_CMD


Make hubber a member of the "shadow" group, capable of accessing the PAM service

	$ sudo usermod -a -G shadow hubber

Test it

	$ sudo -u hubber /opt/hubber/anaconda/bin/python3 -c "import pamela, getpass; print(pamela.authenticate('$USER', getpass.getpass()))"

	>> none

### Create JupyterHub service

Jupyterhub is one of Anaconda's included packages. JupyterLab, IPyWidgets, and wheel are also included.

Create a jupyterhub configuration file

	$ sudo mkdir -p /opt/hubber/etc/jupyterhub/

	$ sudo cd /opt/hubber/etc/jupyterhub/

	$ sudo /opt/hubber/anaconda/bin/jupyterhub --generate-config

	This generated file has many options, all of which are commented out.

Set the "default url" for Jupyterhub sessions to open with the "Lab" interface. The default is "Notebook".

	# Replace the empty string ('') default_url with '/lab'

	sed -i "s/#c.Spawner.default_url = /c.Spawner.default_url = '\/lab' # was /g" jupyterhub_config.py

	c.Spawner.default_url = '/lab' # was ''

The sudospawner extension has this requirement (and two suggestions):

	c.JupyterHub.spawner_class = 'sudospawner.SudoSpawner'

	c.SudoSpawner.mediator_log_level = "DEBUG"

	c.JupyterHub.log_level = 10


#### Sudo non-Root Service

##### Service definition

Create "jupyterhub.service" file within the jupyterhub's "/etc" directory and symbolically link to it from the root /etc/systemd/system/ directory.

	$ sudo mkdir -p /opt/hubber/etc/systemd

	$ sudo nano /opt/hubber/etc/systemd/jupyterhub.service

    copy-and-paste this unit definition


		[Unit]
		Description=JupyterHub
		After=syslog.target network.target

		[Service]
		User=hubber
        WorkingDirectory=/opt/hubber/etc/jupyterhub
		Environment=PATH=/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/opt/hubber/anaconda/bin
		ExecStart=/opt/hubber/anaconda/bin/jupyterhub -f /opt/hubber/etc/jupyterhub/jupyterhub_config.py

		[Install]
		WantedBy=multi-user.target


	$ sudo ln -s /opt/hubber/etc/systemd/jupyterhub.service /etc/systemd/system/jupyterhub.service


Enable the service, run, and check status

	$ sudo systemctl daemon-reload

	$ sudo systemctl enable jupyterhub.service

	$ sudo systemctl start jupyterhub.service

	$ sudo systemctl status jupyterhub.service


Test from a local network computer

	$ firefox http://jupyter:8000



##### Apache2

With Apache2 installed earlier, enable necessary modules. 

	$ sudo a2enmod proxy http_proxy proxy_wstunnel headers expires require

	Instructions elsewhere typically do not include "proxy_wstunnel". It is for the terminal sessions available in labs. Without it, JupyterLab terminals appear to open but are disabled.


Create a configuration file with similar contents. This config has Jupyter Lab as the root URL. 

	$ sudo nano /etc/apache2/sites-available/jupyterhub.conf


	<VirtualHost *:80>

		ServerName jupyterhub.example.com

		ErrorLog ${APACHE_LOG_DIR}/jupyter-error.log
		CustomLog ${APACHE_LOG_DIR}/jupyter-access.log combined

		RewriteEngine On
		RewriteCond %{HTTP:Connection} Upgrade [NC]
		RewriteCond %{HTTP:Upgrade} websocket [NC]
		RewriteRule /(.*) ws://127.0.0.1:8000/$1 [P,L]

	  <Location "/">
	    # preserve Host header to avoid cross-origin problems
	    ProxyPreserveHost on
	    # proxy to JupyterHub
	    ProxyPass         http://127.0.0.1:8000/
	    ProxyPassReverse  http://127.0.0.1:8000/
	  </Location>

	</VirtualHost>



##### SSL with and without Let's Encrypt Certbot

###### using previously-created certificate and key
I recommend running Certbot as part of the installation. Here is a how to just use your previously-created certificate chain and private key files.

When copying letsencrypt certs, remember that in the "live" directory, there are only links to the ../archive directory. And in archive dir, it is a link to a more specifically identified .pem file. For example, "fullchain.pem" is a link to "fullchain.1.pem"

Create a subdirectory in your jupyterhub's "/etc" subdirectory; place the certificate and private key files there and set their ownership to jupyterhub's controlling user, "hubber".

The Apache configuration file for https will have lines similar to the following: 
	...
	SSLCertificateKeyFile /opt/hubber/etc/ssl/privkey.pem
	SSLCertificateFile /opt/hubber/etc/ssl/fullchain.pem
	...

###### Using certbot
To do it right, and having already installed python-certbot-apache, run certbot with the apache option. 

Make sure you have port 80 opened and that DNS points to your remix server's IP. You can close port 80 after certificate installation.

Run certbot. It will generate and install certificates and update Apache2's .conf files.

	$ sudo certbot --apache -m hubber@example.com -d jupyter.example.com


###### Enabling SSL

Edit the apache conf file so that it runs as https only, optionally with permanent redirection.

<VirtualHost *:80>
	ServerName jupyter.example.com 
    Redirect permanent / https://jupyter.example.com
</VirtualHost>

<IfModule mod_ssl.c>
<VirtualHost *:443>

	ServerName jupyterhub.example.com

	ErrorLog ${APACHE_LOG_DIR}/jupyter-error.log
	CustomLog ${APACHE_LOG_DIR}/jupyter-access.log combined


	# configure SSL
	SSLEngine on
	SSLCertificateKeyFile /etc/letsencrypt/live/jupyter.example.com/privkey.pem
	SSLCertificateFile /etc/letsencrypt/live/jupyter.example.com/fullchain.pem
	SSLProtocol All -SSLv2 -SSLv3
	SSLCipherSuite EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH

	# Use RewriteEngine to handle websocket connection upgrades
	RewriteEngine On
	RewriteCond %{HTTP:Connection} Upgrade [NC]
	RewriteCond %{HTTP:Upgrade} websocket [NC]
	RewriteRule /(.*) ws://127.0.0.1:8000/$1 [P,L]

	<Location "/">
		# preserve Host header to avoid cross-origin problems
		ProxyPreserveHost on
		# proxy to JupyterHub
		ProxyPass         http://127.0.0.1:8000/
		ProxyPassReverse  http://127.0.0.1:8000/
	</Location>
  
</VirtualHost>
</IfModule>


Disable the http site and enable the https one.

Fully stop and start apache, waiting a few seconds between commands.

Hit your site to see how it's doing.

Logfiles are your friends



### Alternate Strategies

these can be useful in some situations

#### Install Using Pip

Create a virtual environment and install components "locally".

	$ python3 -m venv /opt/jhub

	$ sudo /opt/jhub/bin/python3 -m pip install wheel

	$ sudo /opt/jhub/bin/python3 -m pip install jupyterhub jupyterlab

	$ sudo /opt/jhub/bin/python3 -m pip install ipywidgets


Create Jupyterhub configuration file

	$ sudo mkdir -p /opt/jhub/etc/jupyterhub/
	
	$ cd /opt/jhub/etc/jupyterhub/

	$ sudo /opt/jhub/bin/jupyterhub --generate-config

Cause Jupyterhub to start sessions in Lab mode, rather than Notebook, by changing the "default_url" in the configuration file.

	# Replace the empty string ('') default_url with '/lab'

	$ sudo sed -i "s/#c.Spawner.default_url = /c.Spawner.default_url = '\/lab' # was /g" jupyterhub_config.py


#### Run Jupyterhub as Root

Create "jupyterhub.service" locally and symbolically link to it from systemd.

	$ sudo mkdir -p /opt/jhub/etc/systemd

	$ sudo nano /opt/jhub/etc/systemd/jupyterhub.service

    copy-and-paste this unit definition

		[Unit]
		Description=JupyterHub
		After=syslog.target network.target

		[Service]
		User=root
		Environment="PATH=/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/opt/jhub/bin"
		ExecStart=/opt/jhub/bin/jupyterhub -f /opt/jhub/etc/jupyterhub/jupyterhub_config.py

		[Install]
		WantedBy=multi-user.target


	$ sudo ln -s /opt/jhub/etc/systemd/jupyterhub.service /etc/systemd/system/jupyterhub.service


Enable service, run, and check status

	$ sudo systemctl daemon-reload

	$ sudo systemctl enable jupyterhub.service

	$ sudo systemctl start jupyterhub.service

	$ sudo systemctl status jupyterhub.service



