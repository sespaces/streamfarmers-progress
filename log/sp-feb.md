# A Streamfarmer's Progress
## February 2020
### Week 1

Several days getting educated about Linux system administration and about Jupyter Hub.


#### Day 1 -- GitLab

[Gitlab Server Setup](./gitlab-installation-ubuntu-18-04.md)
- This was as easy as I imagined. But almost not so. For a while I thought I would have to go throught the long and uncertain process of building it from source. (It has a *lot* of dependencies.)
- Essentially, just use the installer instructions for your system (even if it's a raspberry-pi) from Gitlab's [Install](https://about.gitlab.com/install/) page.
- It even takes care of getting Let's Encrypt certificates.

#### Days 2 - 3 -- Mostly Failed Installations of JupyterLab/JupyterHub

- [Tried and failed](./tljh-fail.md) to install "The Littlest JupyterHub"

#### [Day 5](./sp-feb-05)

- dpkg-reconfigure console-setup

- adduser versus useradd

- enter Anaconda
 
- check the logs -- CTFL, as in RTFM!  It's (not?) funny how often I resist checking the log files when things are going wrong

- ssh known_hosts -- offending keys and strict host key checking

#### [Day 6](./sp-feb-06)

- non-root jupyterhub server dive

- using the "cut" command

- anaconda, jupyterhub, sudospawner installations

- "shadow" group membership

- stuck at running service as root

#### [Day 7](./sp-feb-07)

- systemd service definitions -- if user is not root, you probably want to indicate a working directory other than "/"

- apache2 proxy

- using the "tee" command

- re-using letsencrypt certificates

- finally finished Jupyter Hub setup 

### Week 2

#### [Day 1](./sp-feb-08.md)

- [JupyterHub installation writeup](../jupyter/install-and-run-jupyterhub.md)

- sudo, EH?


#### [Day 2](./sp-feb-09.md)

- [Add Julia kernel to Jupyter Hub](../jupyter/add-julia-kernel-to-jupyterhub.md)

