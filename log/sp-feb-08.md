# A Streamfarmer's Progress (cont'd)

### February 8

- 7:20am

Good morning!

I'm still feeling good about finishing the process of JupyterHub deployment. I'm still writing it up.


##### who do sudo? you do sudo 

I've typed "sudo" a million times but never intentionally used the "-E" option! And rarely have I used "-H".

Working through these server setups using various usernames and with various environments. I finally learned to use them (outside of Python wheel-related installs.

`-E` preserves your environment variables that you might need for executing the command

`-H` preserves only the $HOME variable, preventing installation of configuration and cached files, and the like, from being installed in root's home directory.

- 7:30am

Today I hope to finish writing up the JupyterHub installation how-to.

I'd like also to install the Julia kernel for JupyterHub and to document that process.

- 8:30am

editing streamgardener's progress files while extracting portions for writing how-to

- 9:00am
away
- 11:00am

organize repo and commmit current work

- 11:15am

writing "install-and-run-jupyterhub.md"

- 1:00pm
away
- 4:30pm

writing "install-and-run-jupyterhub.md"


The following snippet came from some failed attempt, but I cannot locate the guide I got it from..

Seems it may help when I move on to installing Julia kernel.


	Once users log in, they can create their own new environments using ipykernel using this command in their own bash shells:

		$ /opt/conda/envs/python/bin/python -m ipykernel install --name 'my-py-env' --display-name "My Python Env"

### Install Julia kernel.


- 6:00pm






