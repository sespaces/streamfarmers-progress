# Add Julia kernel to Jupyter Hub

In short, simply install Julia and add the IJulia package.

#### Install Julia programming language

Julia does not need to be installed system-wide. Simply download, unzip, and run ./bin/julia.

[Download](http://julialang.org/downloads/) the appropriate binaries or source code for your platform and choice of Julia version.

# download Julia 1.3.1 for Linux 

$ wget -O julia131.tar.gz https://julialang-s3.julialang.org/bin/linux/x64/1.3/julia-1.3.1-linux-x86_64.tar.gz

# unzip it and move it into jupyterhub installation directory

$ tar -xzf *.gz

$ mv julia-1.0.5 /opt/hubber/julia

Add the Julia bin directory to the PATH environment variable in the service definition. 

$ nano /opt/hubber/etc/systemd/jupyterhub.service

# edit line beginning with Environment="PATH=..  to include ":/opt/hubber/julia/bin"

$ sudo systemctl daemon-reload

$ systemctl restart jupyterhub

##### Add IJulia in JupyterHub

This method works where JupyterHub users are also system users with home directories. By default Julia's package manager stores files in $HOME/.julia/packages.

Open a terminal as a JupyterHub user and start a Julia interactive session.

	$ julia

	# add IJulia package; close-bracket gets you in, backspace out
	
	julia> ]
	
	(v1.3) pkg> add IJulia
	
	(v1.3) pkg> pre-compile

	(v1.3) pkg> <backspace>

	julia> exit()

	$ exit


Click on File, New Launcher, Jupyterhub should have two new launch icons for Julia Notebook and Julia Console.
