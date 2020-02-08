# Failed Attempt Installing "The Littlest JupyterHub" (tljh) 

Ah, Jupyterlab. What a wonderful bit of software. Using it for Python Notebooks was how I learned about Julia, the super-fast, high-level, functional programming language. 

But JupyterLab is more for a single user, and I want more. JupyterHub provides for multiple users and more. 

There are two types of JupyterHub installations: one for up to a hundred users, and one for more than that. I'm going with the one called "The Littlest JupyterHub," or "TLJH."

Following the [guide from JupyterHub](https://jupyterhub.readthedocs.io/en/latest/) and 
http://tljh.jupyter.org/en/latest/install/custom-server.html

After installing prerequisites, install SSL certificates

    $ sudo certbot cert-only --standalone


Run the tljh installation script. It takes a while

	# curl https://raw.githubusercontent.com/jupyterhub/the-littlest-jupyterhub/master/bootstrap/bootstrap.py | python3 - --admin dhm

Test, reconfigure, generally fail.

Decide that "the hard way" is likely the right way.
