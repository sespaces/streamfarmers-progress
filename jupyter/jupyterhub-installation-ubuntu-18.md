# Jupyterhub Server Installation -- Ubuntu, Apache2, SSL


This how-to is based on the installation instructions available from Jupyterhub, [Install JupyterHub and JupyterLab from the ground up](https://jupyterhub.readthedocs.io/en/latest/installation-guide-hard.html), also known as "the hard way."

It also utilizes an article illustrating how to [run JupyterHub as a non-root user](https://github.com/jupyterhub/jupyterhub/wiki/Using-sudo-to-run-JupyterHub-without-root-privileges).

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


### Install Using Pip

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


### Run Jupyterhub as Root

Create "jupyterhub.service" locally and symbolically link to it from systemd.

	$ sudo mkdir -p /opt/jhub/etc/systemd

	$ sudo nano /opt/jhub/etc/systemd/jupyterhub.service

    copy-and-paste this unit definition


		[Unit]
		Description=JupyterHub
		After=syslog.target network.target

		[Service]
		User=root
		Environment="PATH=/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/opt/jhub/anaconda/bin"
		ExecStart=/opt/jhub/anaconda/bin/jupyterhub -f /opt/jhub/etc/jupyterhub/jupyterhub_config.py

		[Install]
		WantedBy=multi-user.target


	$ sudo ln -s /opt/jhub/etc/systemd/jupyterhub.service /etc/systemd/system/jupyterhub.service


Enable, run, check status

	$ sudo systemctl daemon-reload

	$ sudo systemctl enable jupyterhub.service

	$ sudo systemctl start jupyterhub.service

	$ sudo systemctl status jupyterhub.service




========
Command to be executed by users in their own bash shells if they wish to create their own environments:

	$ /opt/conda/envs/python/bin/python -m ipykernel install --name 'my-py-env' --display-name "My Python Env"









Install Julia kernel.

