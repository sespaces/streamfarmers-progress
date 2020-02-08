


The last time I wrote a shell script was probably a screen menu ".bat" file in the pre-Windows era. Probably MS-DOS 2.x (possibly, but not likely could have even been DR-DOS). How sweet it was to start one of my 5 installed programs with just a press of a number key(!).

But now it's Bash. I need to start a program as a particular user with particular environment settings. There does not seem to be a one-liner that works so let's see what happens

	#!/usr/bin/bash

	su remix

	pm2 start remix-ide

	exit

Hah! and hah! the first is for thinking I could change users and have the script follow me in; it executed "pm2" after I exited user remix; the second is because in exploring the use of environment variables in scripting, I realized that the only thing preventing me from starting remix was not having set the PATH to the bin folder holding all things remixy here "/home/remix/.nvm/versions/node/v10.15.3/bin" 

I rebooted the server to clear all variables I may have set that won't be there on reboot and will attempt to run it from root, make the PATH setting permanent, and then try adding "pm2 start remix-ide" to init.d.


- 10:20am

After reboot, "remix" has lost its path. That makes sense -- I could not find anywhere the cause of remix-ide being in PATH...

So I get to become specific about how things are working.

user remix has all the node_module goodies in her folder; I saved some variables from "env" earlier:

PATH=/home/remix/.nvm/versions/node/v10.15.3/bin:<restofpath>

I think I will just add that to root's and remix's .bashrc files and see if it helpsexport PATH=/home/remix/.nvm/versions/node/v10.15.3/bin:$PATH


Winner, winner, chicken dinner!

so much for the script....