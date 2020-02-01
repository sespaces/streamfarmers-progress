# Remix-IDE Installation -- Ubuntu 18.04 Server and Apache2


## Pre-Installation

- 9:10am (Resizing an LVM)

I've got an ISO for Ubuntu 18.04 Server and 40GB of free disk space.

Or do I have any disk space? Ubuntu's installer wants only to kill whatever is on my disk, as it isn't grokking the LVM already present there. 

I definitely have more to learn about LVM because my ignorance and LVM's attributes have led to frustration and loss. The last time I retried resizing an LVM I destroyed (some useless) partitions.

Not much on this disk needs preserving, but I really hate not knowing how to not waste what has already been done. I will back some directories up then see if I can resize things properly.

With old installation running, I copy and change permissions. Then from my local machine I "scp" the files to myself.

  dhm@solver: $ sudo cp -ra /etc .
  dhm@solver: $ sudo cp -ra /root .
  dhm@solver: $ sudo cp -ra /home/remix/.bash* .

  dhm@solver: $ sudo chown -R dhm:dhm /etc , etc. etc.

  dhm@dhm-laptop: $ scp -r sol:~/root .
  dhm@dhm-laptop: $ scp -r sol:~/etc . etc. etc.

- 10:20am

I will follow the instructions provided in a tecmint.com article (https://www.digitalocean.com/community/tutorials/how-to-use-lvm-to-manage-storage-devices-on-ubuntu-16-04):

$ sudo lvmdiskscan
	...
	0 disks
    7 partitions
    0 LVM physical volume whole disks
    0 LVM physical volumes

????

So Ubuntu server installation disk wants to commandeer the whole disk regardless of what is there? We'll find out more in a bit.

First, we try creating our own LVM on said disk and then run the installatinon again.

$ sudo su

# gdisk /dev/sda
> n        (LVM partition type is '8e00')
> p        (disk partition is sda5)
> w

# lsblk    (no sda5)

# partprobe (or reboot; otherwise new partition doesn't show)

# pvcreate /dev/sda5

# vgcreate solver-VGpool /dev/sda5

# lvcreate -L 40G -n solver-root solver-VGpool

# mkfs.ext4 /dev/solver-VGpool/solver-root

And test it out:
  # mkdir /mnt/solver
  # mount /dev/solver-VGpool/solver-root /mnt/solver
  # echo "'Sup, dog?" > /mnt/solver/sup

Looking good so far!

- 11:15am (

Reboot with 18.04 server installation and again, it wants the whole disk. Geesh.

I would not be using any of the server installation options, and I don't want the desktop minimal installation. I am going to try the "mini" installation ISO. 


http://archive.ubuntu.com/ubuntu/dists/bionic/main/installer-amd64/current/images/netboot/mini.iso

and what a pleasant discovery

it sees all my partitions, including the "spare" 40Gb partition originally sought and mentioned above

it starts minimal, but gives many choices for package groups to install; almost like having lots of installation disks in one!

I choose only "Basic Ubuntu Server" only.

- 12:00pm

It's rebooting, seemingly having preserved the previous distribution.

It booted. Let's see if I can get in from the local network. Connection refused. Falsely assumed "basic server" included openssh-server, but nope.

$ sudo apt install -y openssh-server

Configured it to let me in with my public key and to disallow password authentication.

## Installation

sudo su

apt install -y build-essential apache2 npm

npm install n -g

adduser remix --disabled-login

mkdir /var/remix

chown remix:remix /var/remix

su remix


export REMIXD_COMMIT=eb3d850b06eb05611c97a8b58abc8f940c8e02ce
# latest commit; same as used in successful install

export REMIX_IDE_VERSION=0.9.2
# latest tag; same as used in successful install

export NVM_VERSION=0.35.2
# latest; previous successful was 0.34.0

export NPM_VERSION=6.13.6    
# latest; previous successful was '10.15.3'

export NODE_VERSION=13.7.0
# not really using this variable; just for information

export REMIX_SHARED_DIR=/var/remix
# not used originally, but should this time


# install Node Version Manager
  curl -o- "https://raw.githubusercontent.com/creationix/nvm/v$NVM_VERSION/install.sh" | bash 

# HOME should be user remix's
export NVM_DIR="$HOME/.nvm" 

# load nvm and bash_completion
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
# presumably should work with simple "$NVM_DIR/nvm.sh"
# or "$NVM_DIR/bash_completion" ?
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"
   

# "nvm install $NPM_VERSION" does not work for latest npm on ubuntu
n latest
# gets npm 6.13.2 currently 

npm install remix-ide@$REMIX_VERSION -g 

NO-O-O-O

That's where it goes awry with errors over node-gyp and potentially more...

So, can we use latest remix but previously working npm?

So I f'd up when I installed npm much earlier in process. Rather than fight it; I re-installed ubuntu mini; this time with only ssh-server package






sed -i s/"remixd.git"/"remixd.git#$REMIXD_COMMIT"/g $HOME/.nvm/versions/node/v$NPM_VERSION/lib/node_modules/remix-ide/package.json 

cd $HOME/.nvm/versions/node/v$NPM_VERSION/lib/node_modules/remix-ide 

rm -rf node_modules 

npm install