# A Streamfarmer's Progress (cont'd)

### February 6

	At yesterday's end, I was working through the process of serving Jupyterhub using sudo rather than the root account itself.

	I am using two documents in particular to help guide me.

	One is an [article](https://github.com/jupyterhub/jupyterhub/wiki/Using-sudo-to-run-JupyterHub-without-root-privileges) in jupyterhub's wiki.

	The other is the [example Docker file](https://github.com/jupyterhub/sudospawner/blob/master/examples/Dockerfile) for "sudospawner," also created by jupyterhub on github.com.


	I'll be installing Jupyter using Anaconda's Linux Installer shell script available at https://www.anaconda.com/distribution/



- 6:00am

##### CLI goodies: cut  

Working on the command line is both challenging and rewarding. This morning I took a little time to investigate two commands used in guides.

"cut" excises pieces of text in lines of input; for example, if you want to list the system's user accounts, you need to examine the file "/etc/passwd"; a typical line in this file looks like this: 

	dhm:x:1000:1000:Daniel H. Morgan:/home/dhm:/bin/bash

These fields, delimited by colons, are the account's user name, (password), user id, group id, user full name, home dir, and login shell.

Try "cat /etc/passwd" and you'll see that it's hard to read. ("getent passwd" is equivalent). 

With "cut," you can have just the fields you want. For example, if you want just users' name and full-name fields, the command is:

	$ cut -d: -f1,5 /etc/passwd
	>> ... (lots of other accounts)
	>> dhm:Daniel H. Morgan

In that example,

	"-d:" -- the delimiter between pieces is the ":" character; default is a "tab"

	"-f1,5" -- select fields 1 and 5


- 7:45am

Create a user to control Jupyterhub

	$ sudo adduser hubber


Create a group for jupyterhub users and add our users

	$ sudo addgroup jupyterhub

	$ sudo usermod -a -G jupyterhub hubber

	$ sudo usermod -a -G jupyterhub dhm


### Install Anaconda

Create a directory on /opt for anaconda, with hubber as the owner

	$ sudo mkdir /opt/hubber

	$ sudo chown hubber:hubber /opt/hubber

Get and run the installation script. Note that the version is "2019.10" does not refer to the Ubuntu version and will work just fine. Also, do not use "sudo" even though their instructions suggest it. Do use "bash" even if you are already in a bash shell. Script will prompt for location; I entered "/opt/hubber/anaconda". I also selected automatic activation and to "conda init" at end of process.

	$ wget https://repo.anaconda.com/archive/Anaconda3-2019.10-Linux-x86_64.sh

	$ bash Anaconda3-2019.10-Linux-x86_64.sh

The installation modifies the user's .bashrc, including adding the virtual-environment indicator, "(base)", to the shell prompt.

I have a customized prompt and the installation kind of messed it up. In .bashrc, I moved my "PS1= ..." line to the end and placed $CONDA_PROMPT_MODIFIER in the spot where I want "(base)" to appear.

Jupyterhub users will have shell access, so make everyone's home directory inaccessibe to "others" 

	$ sudo chmod o-rwx /home/*


- 10:00am
break
- 10:30am

Install the "sudospawner" program from jupyterhub

	(base) $ pip install sudospawner

Configure sudo to allow hubber to use sudospawner on behalf of other jupyterhub users.

Create a sudo configuration file in /etc/sudoers.d/jupyterhub-sudo-conf with three lines. (see README in that directory for information)

	$ sudo nano /etc/sudoers.d/jupyterhub-sudo-conf

Copy and paste therein these lines:                      

	# an alias allows password-free usage   
	Cmnd_Alias SPAWNER_CMD = /opt/hubber/anaconda/bin/sudospawner

	# allow any jupyterhub member to spawn single-server as hubber
	%jupyterhub ALL=(hubber) /usr/bin/sudo
	hubber ALL=(%jupyterhub) NOPASSWD:SPAWNER_CMD

- 12:50pm

Well, alrighty, then. Seems I now have things working at least this far: any member of jupyterhub is able to run sudospawner (and nothing else) with no password


Make hubber a member of the "shadow" group, capable of accessing the PAM service

	$ sudo usermod -a -G shadow hubber

Test it

	$ sudo -u jhub /opt/jhub/anaconda/bin/python3 -c "import pamela, getpass; print(pamela.authenticate('$USER', getpass.getpass()))"

	>> none

- 1:10pm
- 4:00pm

Install Apache2

	$ sudo apt install -y apache2


Install Certbot

	$ sudo add-apt-repository ppa:certbot/certbot

	$ sudo apt update && apt install -y python3-certbot-apache

Install NodeJS and NPM. (old, but not out-dated for Jupyterhub)

	$ sudo apt install -y nodejs npm

Install configurable-http-proxy for Node

	$ sudo npm install -g configurable-http-proxy


### Install JupyterHub

I ran these three commands for no reason; all were installed with Anaconda

	$ sudo /opt/jhub/anaconda/bin/python3 -m pip install wheel

	$ sudo /opt/jhub/anaconda/bin/python3 -m pip install jupyterhub jupyterlab

	$ sudo /opt/jhub/anaconda/bin/python3 -m pip install ipywidgets

Create a jupyterhub configuration file

	$ sudo mkdir -p /opt/jhub/etc/jupyterhub/

	$ sudo cd /opt/jhub/etc/jupyterhub/

	$ sudo /opt/jhub/anaconda/bin/jupyterhub --generate-config

This configuration file has many (but not all) options, each of which is commented out.

Set default url to open in Jupyterhub sessions as Lab, rather than as a Jupyter Notebook by changing the "default_url" in the configuration file.

	# Replace the empty string ('') default_url with '/lab'

	sed -i "s/#c.Spawner.default_url = /c.Spawner.default_url = '\/lab' # was /g" jupyterhub_config.py

Other changes to the generated config file:

	# find every set configuration line (they begin with a "c."

	$ grep '^c.' /opt/jhub/etc/jupyterhub/jupyterhub_config.py
	c.JupyterHub.spawner_class = 'sudospawner.SudoSpawner'
	c.SudoSpawner.mediator_log_level = "DEBUG"
	c.JupyterHub.log_level = 10
	c.Spawner.default_url = '/lab' # was ''



Create "jupyterhub.service" locally and symbolically link from systemd.

	$ sudo mkdir -p /opt/hubber/etc/systemd

	$ sudo nano /opt/hubber/etc/systemd/jupyterhub.service

    copy-and-paste this unit definition


		[Unit]
		Description=JupyterHub
		After=syslog.target network.target

		[Service]
		User=root
        WorkingDirectory=/opt/hubber/etc/jupyterhub
		Environment=PATH=/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/opt/hubber/anaconda/bin
		ExecStart=/opt/hubber/anaconda/bin/jupyterhub -f /opt/hubber/etc/jupyterhub/jupyterhub_config.py

		[Install]
		WantedBy=multi-user.target


	$ sudo ln -s /opt/jhub/etc/systemd/jupyterhub.service /etc/systemd/system/jupyterhub.service


Enable, run, check status

	$ sudo systemctl daemon-reload

	$ sudo systemctl enable jupyterhub.service

	$ sudo systemctl start jupyterhub.service

	$ sudo systemctl status jupyterhub.service


Test from a local network computer

	$ firefox http://10.10.100.11:8000

- 5:45pm

OK. That is working. BUT....

Currently the service is running as root, not hubber. If I change the Unit definition; it has permission errors. The first was the "cookie_secret" file, so I hard-coded its location in the config file. It then erred out on the next line requiring file access.






===============



 Configuration file for jupyterhub.

c = get_config()

# use the sudo spawner
c.JupyterHub.spawner_class = 'sudospawner.SudoSpawner'
c.SudoSpawner.mediator_log_level = "DEBUG"
c.JupyterHub.log_level = 10









