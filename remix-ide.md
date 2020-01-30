A Streamgardener's Progress
===========================

## Remix-IDE

### Installation
    
   *not* an easy task; more difficult if you are newish to world of nodejs
   
   many errors and failed attempts using ubuntu 18.04...; will try again after [cross fingers here]  success with :
    
   ubuntu 16.04 server + lightdm , nodej10.6/npm6.13, 
    
   git clone remix-ide 
   
   curl localhost:8080

##### 2020, 28 January

- 7:30am (status)

    best result thus far is to serve index.html that fails to load "./build/app.js"
    
    the file is there and permissioned correctly but fetch attempt yields 404 error
  
  
- 8:30am (implement X11 forwarding)

    working on this issue brings up another issue for me: how to remotely use a web-browser to view "localhost" content from the server

    X11 Forwarding is the answer

    I haven't used it before, and it took a little delving, but I now have it working

    requires two steps, one each for server and client

    on server: in /etc/ssh/sshd_config, set "X11Forwarding yes" and reload sshd

    on client: in ~/.ssh/config (create if necessary), add "ForwardX11 yes"

  9:15am (x11forwarding chromium-browser installations mismatched)

    so X11Forwarding is partially successful; SublimeText works, but Chromium Browser not so much; seems client and server are running different versions....

    installed firefox and it works

    server's localhost comes up, but not the 8080...

  10:40am (getting to know npm)

    looking at log files from failing remix-ide service; have tried several methods from several angles each; time to understand the node-js package management system

    its power and simplicity are very impressive; very tight; reading the help files makes me feel like I am getting smarter just by seeing their form

    I think I have now just uninstalled everything remix-ide related
    $ sudo nmp uninstall -g -S -D remix*


    had earlier apt installed git-all

    thanks to some documentation along the way, I am checking travis-cl to learn which versions of things remix passed on

    https://circleci.com/gh/ethereum/remix-ide

    which leads to same for remix-live which says that: NOT PASSING! fuck; oh, wait, it is not configured for testing directly; the passing example in the main remix-ide includes remix-live along with it and it passes...

    I see in the configuration file for the build tester that they are using node version 9.11.2 and not the latest nor 10.2 as someone else suggested.....

    $ sudo n 9.11.2
    ...: installed : v9.11.2 (with npm 5.6.0)

and the frustration continues; yes I know a little more than I did before, but fuck




##### 2020, 29 January 

- 8:10am (regroup and refresh)

  now I know a bunch more, having spent a good part of yesterday and the early part of this morning working on the issue and on regrouping in preparation for another attack upon it (sounds too vicious; what an ugly descriptor -- stop; breathe.)

  part of regrouping has been to tidy up my workspace a bit; here is something I like to add to the end of my .bashrc; the "ec" alias is for "edit [bash]configuration" and just makes it easier to tweak the settings
    ...
    alias ec='nano ~/.bashrc && source ~/.bashrc'
    export PS1="\n\n\l:\!   \h:\u    \w     \@ \n\$ "
    ...

  which produces a prompt and output like this:

    0:2006   dhm-laptop:dhm    ~/current/progress     08:49 AM 
    $ subl remix-ide.md


    0:2008   dhm-laptop:dhm    ~/current/progress     08:59 AM 
    $

  two blank lines precede each new prompt, giving separation from prior output; the session number (\l) is 0, meaning this first tty connection; the line number in bash history (\!) is 2008, which is helpful when repeating commands; host (\h) and user (\u) names I usually leave out but currently I am switching between machines and working as root too often to keep track; current working directory (\w) and time (\@) are helpful; 

  what was command 2007 ?
    $ !2007
      tail -4 ~/.bashrc
      alias ec='nano ~/.bashrc && source ~/.bashrc'
      export PS1="\n\n\l:\!   \h:\u    \w     \@ \n\$ "


  stop typing (and so often incorrectly) the ip's of the machines and servers on the network by giving them names

    $ sudo nano /etc/hosts
      ...
      127.0.1.1 solidity solidity.fvill.com remix.fvill.com 
      10.10.100.8 ethel
      10.10.100.3 dhm-laptop lap
      10.10.100.2 dhm
      ...

    now this works:

      $ ssh lap



  make server console displays readable; default size is 8x12 which is too small on screens with decent resolutions

    $ sudo dpkg-reconfigure console-setup

    [my answers: utf-8, latin-slavic-greek, TerminusBold, 12x24]

- 10:10am (back to

having multiple areas of ignorance makes it difficult to troubleshoot




- 1:00pm

I have made a new installation of ubuntu 18.10 server with no pre-selected packages; I've run 'apt update' and 'apt upgrade'; sshd is currently allowing me in with password but will accept public key; .bashrc has minor changes as above; also /etc/hosts

I was seemingly having good luck following the installation layout for the Remix-IDE docker file. Until I ran out of disk space. I think that is fixed now.

The original docker file is at:

    $ git clone https://www.github.com/4c0n/remix-ide-docker

I went through the steps, altering for executing on system through shell.

[Installing Remix-IDE](./remix-ide-bash-installation)

This time I made it all the through; and gosh I feel like a big boy. 

- 2:30pm (how now)

I went to push these notes and instructions to github but I haven't yet uploaded a key from this machine.

I am pretty sure you have to do that through browsing their site.

Doh. and also, make sure you didn't clone your repo originally using "https://"; otherwise github thinks you think you are special; 
$ git remote set-url origin "git@github.com:my-id/my-repo.git"

4:45pm (solver.fvill.com:443)

As happy as I am to have remix serving on :8080, I need it accessible via https; and if I can't set it to use my public key, it needs at least to have a username:password access method

I expect to run the remix server on http behind apache on https

Once I get the proxy business worked out, I'll add password authentication, then the SSL layer with letsencrypt certificates

5:00pm (limit ssh access to public key only)

yikes! sshd is still open to passwords and even though it faces public on non-standard port, this ain't good

  $ sudo sed  -i s/"PasswordAuthentication yes"/"PasswordAuthentication no"/g /etc/ssh/sshd_config

  $ sudo service sshd reload

  $ sudo ssh -p 22222 dhm@solver.fvill.com

    dhm@solver.fvill.com: Permission denied (publickey).

5:10pm (now, about that proxy)

make sure it isn't running since I think it isn't installed

  $ sudo service apache2 status

  $ sudo apt install -y apache2 apache2-utils apache2-doc

  $ sudo reboot now

doh! can't ssh in...

  log in to physical remix-ide server box

  open sshd to password auth

  get back on two other network machines and ssh-copy-id's more better than before...

  shut sshd to password auth

  logging in with public key from both machines, locally and round-tripping the internet

5:50pm (now about that proxy, er, that remix.service)

Apache2 is serving at localhost

I will pause configuring a site because it seems more essential to get the remix-ide server running as a service that starts at machine boot

First, add a node process manager:
  # as user "remix"
  $ npm install pm2@latest -g

Next, learn that our remix-ide can be started with a straight "remix-ide" and it goes to right folder and port.

Crank it up with process manager:

  $ pm2 start remix-ide

  output says it's online, and it is


Add it to machine's init file? And does it need to be started by user "remix"?

9:00pm

well, I'm kind of stuck again

having difficulty getting service to start because the environment is a bit tricky; I can install and run as user remix but not get the startup automated; I cannot avoid that challenge by installing as user "root" using the same method

good night

- 6:45am (trek on, little buddy)

Having the appearance of my terminal windows reflect the session's connection and user role. I am using Gnome Terminal and have six "profiles" I use to distinguish normal and root user across three machines. All profiles have the same column and row counts and font size. Each machine has an easily distinguishable color pallette. The "root" profile for each has a more intense background color, bold font, and changed cursor shape.

I had hoped to find and share some configuration file information, but the last 15 minutes taught me that gnome-terminal, along with other gnomey stuff, has a binary registry file that can be accessed via "dconf-editor" (or not?; I installed it and had no luck there either). Harumpf; not very typical.

time to move on

- 7:30am

Situation review:

  seems I need to understand more about node...

  goal: serve remix-ide at https://solver.fvill.com

  steps: 

    fresh install doing everything as root

    or

    write script that launches it properly and can be called at system initialization; not as root

  seems I have some bash scripting education on the plate


  goal-step-1: use a script to launch remix

  goal-step-2: cause the script to be run at reboot

  goal-step-3: serve remix through mod_proxy

  goal-step-4: serve remix via https


10:30am (launched from root)

see [bash-scripting](./bash-scripting.md)

Step 1 -- Done 

  root now launches remix-ide via pm2

  underlying problem had been that I gave user remix a password and ability to login; proper method is `$ sudo adduser --disable-login remix`; resolution is `$ sudo usermod --expiredate 1 remix` which causes their password to expire

  this limits access to account to "sudo su remix"

  additionally, the PATH env variable needs to include the bin directory with all the components; I added it to root's .bashrc

    `$ sudo echo "export PATH=/home/remix/.nvm/versions/node/v10.15.3/bin:$PATH" >> ~/.bashrc`

- 10:45am

Step 2 
