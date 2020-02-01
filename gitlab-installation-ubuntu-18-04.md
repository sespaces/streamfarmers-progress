## Gitlab Installation

This was as easy as I imagined. But almost not so. For a while I thought I would have to go throught the long and uncertain process of building it from source. (It has a *lot* of dependencies.)

Essentially, just use the "Omnibus" installer instructions for your system (even if it's a raspberry-pi) from Gitlab's [Install](https://about.gitlab.com/install/) page.

It even takes care of getting Let's Encrypt certificates.

What you get is the same software as Gitlab Enterprise Edition, just limited in functionality to that of the Community Edition. And the "CE" includes a *lot* of functionality.

#### Preparation

In order to install the Let's Encrypt certificates automatically, make sure that your site is accessible via ports 80 and 443 for this process. Port 80 may be closed after installation.

Your machine should also have a fully qualified domain name.

Install the pre-installation package dependencies if needed.

	$ sudo apt install -y curl openssh-server ca-certificates

Install postfix for email notifications or configure for use with another mail server. This isn't as complex as I feared, since it isn't a complete mail system.

	$ sudo apt install -y postfix postfix-doc

	answer "internet site," your domain name, and that was it for me; all else was automatic or default values

Get and run the installation setup script for your particular Linux distribution.

	$ curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.deb.sh | sudo bash

Install Gitlab.

	$ sudo EXTERNAL_URL="https://my-gitlab-server.example.com" apt-get install gitlab-ee

Edit the Gitlab configuration file as desired. I elected to have Gitlab manage my ca-certs so I went to the letsencrypt section of the "gitlab.rb" configuration file.

	$ sudo nano /etc/gitlab/gitlab.rb

		letsencrypt['enable'] = true
		
		letsencrypt['contact_emails'] = ['dhmorgan@protonmail.com']

		letsencrypt['auto_renew'] = true

Always reconfigure after making changes

	$ sudo gitlab-ctl reconfigure

Confirm that it works.

Shut down port 80.

Enjoy!