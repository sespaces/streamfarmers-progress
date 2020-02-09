# SFP Log 2020 Feb 09

Sunday morning; field station 1

- 7:15am
The draft for the jupyterhub installation how-to is done.

I would like to run through the guide on another machine to help remove any serious flaws and perhaps add any further pointers.

But adding the Julia kernel awaits. And that was going to be next. And it seems more interesting. 

### Adding the Julia kernel to JupyterHub

Documentation on this topic is scant and outdated.

Gist is to install Julia and IJulia.

#### Install Julia programming language

Julia does not need to be installed system-wide. Simply download, unzip, and run ./bin/julia.

[Download](http://julialang.org/downloads/) the appropriate binaries or source code for your platform and choice of Julia version.

The latest stable and LTS releases are versions 1.3.1 and 1.0.5, respectively.

I grabbed them both.

	$ sudo mkdir -p /opt/julia/zips

	$ sudo chown -R hubber:hubber /opt/julia

	$ cd /opt/julia/zips

	$ wget -O julia131.tar.gz https://julialang-s3.julialang.org/bin/linux/x64/1.3/julia-1.3.1-linux-x86_64.tar.gz

	$ wget -O julia105.tar.gz https://julialang-s3.julialang.org/bin/linux/x64/1.0/julia-1.0.5-linux-x86_64.tar.gz

	$ tar -xzf *.gz

	$ mv julia-1.0.5 ../1.0

	$ mv julia-1.3.1 ../1.3

Testing 1 3 1

	$ /opt/julia/1.3/bin/julia
	               _
	   _       _ _(_)_     |  Documentation: https://docs.julialang.org
	  (_)     | (_) (_)    |
	   _ _   _| |_  __ _   |  Type "?" for help, "]?" for Pkg help.
	  | | | | | | |/ _` |  |
	  | | |_| | | | (_| |  |  Version 1.3.1 (2019-12-30)
	 _/ |\__'_|_|_|\__'_|  |  Official https://julialang.org/ release
	|__/                   |

	julia> exit()

Move or copy the installation directory into Jupyterhub's installation directory.

	$ cp -Ra /opt/julia/1.3 /opt/hubber/julia


Add Julia bin directory to the PATH environment variable in the service definition.

	$ nano /opt/hubber/etc/systemd/jupyterhub.service

	$ sudo systemctl daemon-reload

	$ systemctl restart jupyterhub

##### Add IJulia JupyterHub

IJulia is also easy to install and is done on a per-user basis.

This method works for my installation, where JupyterHub users are also system users with their own home directories. By default Julia's package manager stores files in $HOME/.julia/packages.

Open a terminal as a JupyterHub user and start a Julia interactive session.

	$ julia

	# add IJulia package; close-bracket gets you in, backspace out
	
	julia> ]
	
	(v1.3) pkg> add IJulia
	
	(v1.3) pkg> pre-compile

	(v1.3) pkg> <backspace>

	julia> exit()

	$ exit


By the time you can click on File, New Launcher, Jupyterhub should have two new launch icons for Julia Notebook and Julia Console.


#####

I spent some time unsuccessfully trying to get two versions of Julia to run at once in this setting. Will have to let that go for now. 

- 11:45am
writing up [this process](../jupyter/add-julia-kernel-to-jupyterhub.md)

commit and push

- 12:15pm


