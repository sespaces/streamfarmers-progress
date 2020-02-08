# A Streamfarmer's Progress



February

Week 1

#### Tuesday, February 5 (JupyterLab; console-setup...

- 7:00am

Today I'll be installing JupyterLab. The past couple of days I have been setting up a JupyterHub server. I finally got it it functioning properly except for one feature, the terminal session. 

Solving the issue is made more challenging because JupyterHub is simply a multi-user- (and optionally, a multi-location-) JupyterLab server environment.

Toward the end of my last forray in this area, I found a JupyterHub document indicating that it should be run as non-root. I had originally thought that, as well, but all of the main documentation for installation simply uses root.

After I get the no-terminal-issue resolved, I will redo the installation in the proper way and write a proper how-to.

##### console-setup

But first, a quick side trip into "console-setup" and Ubuntu server. previously changed the font size in the server's console using "dpkg-reconfigure console-setup." But on rebooting, it reverts back to the old font. Apparently, there is a bug in Ubuntu server that causes it to transpose the font height and width values.

The simplest solution seems to be editing "/etc/default/console-setup" file, merely flipping the values in the FONTSIZE variable. The change should persist until the next reconfigure.

Alternate solutions are out there, such as renaming the font in /etc/console-setup/ directory, again by transposing height and width.

#### jupyterlab installation

7:30am

Well, alrighty then. Seems "--disabled-login" is not an "useradd" option on this server. It works on my laptop.... Running "uname -r" on the server returns "4.15.0-76-generic"; on the laptop, "5.3.0-7625-generic." Those are 18.04 and 19.10 kernels.

Perhaps "useradd -r ..." and make it a system user? Not likely since it doesn't do groups. Better is "useradd" -e 1 <username>" which backdates the expiration of the user's password to the first second of the Unix epoch.

Doh! So I just learned the difference between "useradd" and "adduser." The latter is to be preferred as it is friendlier and provides more options. "Adduser" is a front-end for "useradd". Similarly with "deluser" and "userdel," fwiw. (I had apparently used "adduser" on my laptop and "useradd" on server.)

	$ sudo adduser --disabled-login jlab

	$ cd /home/jlab

	$ python3 -m venv jlab

	$ source jlab/bin/activate

	(jlab) $ pip3 install --user jupyterlab

	(jlab) $ export PATH=$PATH:/home/jlab/.local/bin

	(jlab) $ jupyter notebook --version
		   >> error: module jupyter_core not found


- 8:30am

#### Anaconda

Since that "pip install" attempt failed, I will try the "just install Anaconda and you will have JupyterLab" approach. It's been a long time since I experimented with Anaconda, so let's see.

I do not; but if you want GUI apps, install pre-requisites:

	$ sudo apt install libgl1-mesa-glx libegl1-mesa libxrandr2 libxrandr2 libxss1 libxcursor1 libxcomposite1 libasound2 libxi6 libxtst6

Download and run the installation script; use 'bash' command, said the instructions (https://docs.anaconda.com/anaconda/install/linux/).

	$ wget https://repo.anaconda.com/archive/Anaconda3-2019.10-Linux-x86_64.sh

	$ sudo bash Anaconda3-2019.10-Linux-x86_64.sh

Anaconda is now installed at /opt/anaconda and my .bashrc was set to activate the "base" anaconda environment.

Did it work?

	(base) $ jupyter notebook --version
	>> 6.0.1

	(base) $ jupyter lab --version
	>> 1.1.4

Running -notebook  and -lab both start up and indicate they are being served. But when I try to access them through the browser, I keep getting connection-error....

Maybe Apache and my jupyterhub.service are to blame?

	$ sudo systemctl stop jupyterhub

	$ sudo systemctl stop apache2

	$ jupyter notebook

	Browser opens on my remote machine; cannot find file

Maybe there's something funky going on through the shell? I have been remotely opening Firefox via ssh.


Hah! just learned you can switch to tab-N in firefox using alt-<number>; that's handy.


I just installed the light-weight display manager, "lightdm," and started it up. But it won't let me log in. F'ing weird. What did I forget?

And another hour and half and I have got nothing.

Install Gnome display manager (gdm3).

Two minutes later and I can log in through gdm. Fuck lightdm (on ubuntu).

Both "notebook" and "lab" open on the server's display. Lab includes working terminal sessions.

some relief; some frustration and disapproval 

- 10:30am

Fiddling about with things and now we see that JupyterHub also opens up on local server. When accessed directly via localhost:8000, it gives me a terminal. But if through apache proxy, nope.

At least we know that much now.

It shows that what is sticking is the proxy passing.



- 11:00am

did a lot of hacking at jupyterhub config, mostly changing ip's and bind_urls around, and proxy settings in apache

all to no avail 

output from systemctl status indicates the issue may really be with PAM, which means that it's time to dig into authentication

- 1:20pm

but first I need to check one more apache configuration thing; that didn not help

google PAM cannot set UID error...

modify /etc/pam.d/login, commenting out require uid.so line

rebooted machine to clear its head

nope

I'm remembering something I saw earlier about multi-user environments and group membership to help

- 3:00pm

that didn't help, so far

I am getting helpful messages now, I suppose, since set a proper log file.

	$ tail /var/log/apache2/jupyterhub-error.log

	>> No protocol handler valid for /user/dhm/terminals/websocket/3

CHA-CHING!!!!!!!

	$ sudo a2enmod proxy_wstunnel

Thank you, "willingc" for providing the answer (https://github.com/jupyterhub/jupyterhub/issues/367#issuecomment-243461712).

##### systemctl disable

after I installed gdm3 on server, it starts automatically at reboot, which is undesirable

	$ sudo systemctl stop gdm3

	$ sudo systemctl disable gdm3

It still starts when I want to use it

	$ sudo systemctl start gdm3


- 3:20pm

time to go


- 5:45pm

It looks like I should be able to just enable proxy_wstunnel and have my other installation's otherwise-fully-functional JupyterHub service working properly. But I already damaged it with fix-it attempts. Plus I want to set this up using a non-root user.

I have now installed a fresh Ubuntu server 18.04. During install I elected to install openssh-server and to import my github key, leaving "allow password ssh" unchecked; I also selected no packages to install.



##### ssh host id key conflict resolution

during development I keep ssh'ing into the other machine which is being booted up from ever-changing file-systems

that leads to this 

	$ ssh jupyter

	@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
	@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
	@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
	IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
	...
	...
	...
	Offending ECDSA key in /home/dhm/.ssh/known_hosts:37
	  remove with:
	  ssh-keygen -f "/home/dhm/.ssh/known_hosts" -R "jupyter"
	ECDSA host key for jupyter has changed and you have requested strict checking.
	Host key verification failed.


I suppose I should turn off strict checking (maybe I will), but I usually think that I'd rather not take the chance of not turning it back on. 

So I delete the offending line, and its next one because it is also offensive.

	$ sed -i '37,38d' ~/.ssh/known_hosts

<<< update (feb.8) >>>> 

[strict checking](https://serverfault.com/a/559886)

Turn it off momentarily with:

	$ ssh -o StrictHostKeyChecking=no 

Turn it off permanently on a per-host basis by adding the following to your ssh config file

	Host <your problematic host>
       StrictHostKeyChecking no
	
	$ echo -e "Host <offending-host> \n StrictHostKeyChecking no \n" > ~/.ssh/config

(note: I really enjoyed using that "-e" there; I've tried getting line returns to work with echo before, but just gave up, believing it wasn't an option)

<<< end of update >>>> 



- 6:00pm

Bring a new server into the world

I can log in with no password, as hoped. 

	$ ssh jupyter

Fix console font with 
	
	$ sudo dpkg-reconfigure console-setup
    
    $ sudo nano /etc/default/console-setup

(It's still fun to watch the server's console screen change when I make changes to it from my laptop.)


Install some required components

	$ sudo apt install 


configurable_http_proxy

spawnsudo

Create a system user, hubber


Create a directory for JupyterHub and give hubber ownership.


Install Anaconda just for hubber


Install Apache2





https://github.com/jupyterhub/jupyterhub/wiki/Using-sudo-to-run-JupyterHub-without-root-privileges








 Configuration file for jupyterhub.

c = get_config()

# use the sudo spawner
c.JupyterHub.spawner_class = 'sudospawner.SudoSpawner'

c.SudoSpawner.mediator_log_level = "DEBUG"
c.JupyterHub.log_level = 10





# whitelist of users that can spawn single-user servers
Runas_Alias JUPYTER_USERS = io, europa, ganymede, callisto, rhea

# the command(s) jupyterhub can run on behalf of the above users without needing a password
# this command handles the signalling of spawning servers
Cmnd_Alias SPAWNER_CMD = /opt/conda/bin/sudospawner

# actually give hub user permission to run the above command on behalf
# of the above users without a password
rhea ALL=(JUPYTER_USERS) NOPASSWD:SPAWNER_CMD






