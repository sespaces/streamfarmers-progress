# A Streamfarmer's Progress (cont'd)

### February 7

- 5:30am

I woke up this morning and started wondering why I keep permission errors when launching jupyterhub.service with 'user=hubber' in the unit definition. 

My first thought was that maybe there isn't information about the issue because it shouldn't be an issue if it is running behind the web server and port 8000 is only accessible locally. 

I web-searched 'systemd non-root permissions' to find out and DOH! I need to put the WorkingDirectory directive in the service definition. Obviously very obvious to anyone who's messed about in this area. 

I could see from the errors (no permission for "/secret_cookie" implied that it expected the secret to be in root folder) that it wasn't running in the right space. But I had searched for answers using "CWD" as the term, not "WorkingDirectory".

In any event, I think I will have this completed sooner rather than later.

#### jupyterhub.service file


		[Unit]
		Description=JupyterHub
		After=syslog.target network.target

		[Service]
		User=hubber
	***	WorkingDirectory=/opt/hubber/etc/jupyterhub
		Environment=PATH=/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/opt/hubber/anaconda/bin
		ExecStart=/opt/hubber/anaconda/bin/jupyterhub -f /opt/hubber/etc/jupyterhub/jupyterhub_config.py

		[Install]
		WantedBy=multi-user.target


6:30am

[drats; I just closed this file accidentally without saving it; here are some pieces as I recall]

### Apache2 proxy

Apache is installed already; enable necessary modules

	$ sudo a2enmod proxy http_proxy proxy_wstunnel headers expires require

Create a configuration file with these contents

	$ sudo nano /etc/apache2/sites-available/jupyterhub.conf

	<VirtualHost *:80>

		ServerName jupyterhub.example.com

		ServerAdmin me@example.com
		#DocumentRoot /var/www/html

		#LogLevel debug ssl:warn

		ErrorLog ${APACHE_LOG_DIR}/jupyter-error.log
		CustomLog ${APACHE_LOG_DIR}/jupyter-access.log combined

		ProxyRequests Off
		ProxyPreserveHost On
		ProxyVia Full

		<Proxy *>
		Require all granted
		</Proxy>

		ProxyPass / http://127.0.0.1:8080/
		ProxyPassReverse / http://127.0.0.1:8080/

	</VirtualHost>


- 7:15am a cup of tee

My current configuration, as described above, is not working properly. "/" brings up default page and :8000 brings up jupyterhub -- no proxying happening.

I hate seeing the default page because I don't know where it's coming from sometimes. 

I attempted to overwrite it with my preferred starter test page that looks like this:
	<html>
	  <head>
	  	<title>jupyterhub: 'Sup, dog?</title>
	  </head>
	  <body>
	  	<h2>'Sup, dog?</h2>
	  </body>
	</html>

I tried this:

	$ sudo echo "<html><head><title>jupyterhub: 'Sup, dog?</title></head><body><h2>'Sup, dog?</h2></body></html>" > /var/www/html/index.html

But it gives me a permission-denied error. But I'm sudo-ing!

Seems "echo" output cannot be directed directly, or rather, the sudo permissions do not persist.

The answer is to pipe it through "tee", with "sudo" on the other side of the pipe from "echo".

	$ echo "<html><head><title>jupyterhub: 'Sup, dog?</title></head><body><h2>'Sup, dog?</h2></body></html>" | sudo tee /var/www/html/index.html


- 8:00am
#### allow IPv4

Now that that is done, I keep getting 503 errors when I attempt to hit the site.

"netstat -tulpn" shows that port 80 is only bound with IPv6

Seems I may previously have had to go into apache's root http.conf and comment out its Listen so that I could add it to my own? 


DOH! don't need no IPv4; that wasn't the issue; in fact rarely is it even needed these days; Ubuntu is presumably doing some mapping in the background (?)


after a bit more learning about apache2 configs and such; I check the error log and it is yelling at me that I don't have no service running on :8000, which is what apache2 wants when I ask for "/"....

start jupyterhub; reload apache; half-sigh of relief

when you make as many mistakes I do, it pays to get an early start

jupyterhub server is now being proxied

but is still available on port :8000 ??

I see I don't have my re-write rules in the conf file...

I addded the rewrite rules and restarted but still available

The outside world can only access port 443, and it is good that I can get to it from local network. 

Leaving as is.

- 8:30am

- 12:15pm

### Implement SSL

I have certificates already but I am interested in seeing what jupyterhub will do about managing that from within, if possible.

"The Littlest JupyerHub" has this feature, implemented through its "tljh-config set https.enabled true" command.

On the other hand, this installation has SSL implemented beforehand via Apache2 configuration.

So, do usual Apache certbot..., or not. Regardless, once certbot is installed, I could run

	$ sudo certbot --apache -m me@example.com -d remix.mydomain.com

But I don't want to open and close ports on router and I do want to feel more confident about copying certs and keys over from previous installs.

Normally, there are extra benefits to using certbot that probably make it more than worth it to use. Since this install is temporary, I will let those benefits go.

When copying letsencrypt certs, remember that in the "live" directory, there are only links to the archives directory. And in archive dir, it is a link to a more specifically identified .pem file.

This copying is being done from an installation on a different disk partition, currently mounted.

	$ cd /opt/hubber/etc/ssl

	$ sudo cp /mnt/etc/letsencrypt/archive/domain.com/fullchain1.pem fullchain.pem

	$ sudo cp /mnt/etc/letsencrypt/archive/domain.com/privkey1.pem privkey.pem

	$ sudo chown hubber:hubber ./*

Edit the apache conf file so that it runs as https only.

<VirtualHost *:80>
	ServerName jupyter.fvill.com 
    Redirect permanent / https://jupyter.fvill.com
</VirtualHost>

<IfModule mod_ssl.c>
<VirtualHost *:443>
	...
	...
	...

  # configure SSL
  SSLEngine on
  SSLCertificateKeyFile /etc/letsencrypt/live/jupyter.fvill.com/privkey.pem
  SSLCertificateFile /etc/letsencrypt/live/jupyter.fvill.com/fullchain.pem
  SSLProtocol All -SSLv2 -SSLv3
  SSLCipherSuite EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH

  # Use RewriteEngine to handle websocket connection upgrades
  RewriteEngine On
  RewriteCond %{HTTP:Connection} Upgrade [NC]
  RewriteCond %{HTTP:Upgrade} websocket [NC]
  RewriteRule /(.*) ws://127.0.0.1:8000/$1 [P,L]

  <Location "/">
    # preserve Host header to avoid cross-origin problems
    ProxyPreserveHost on
    # proxy to JupyterHub
    ProxyPass         http://127.0.0.1:8000/
    ProxyPassReverse  http://127.0.0.1:8000/
  </Location>
  
</VirtualHost>
</IfModule>


Disable the http site and enable the https one.

Fully stop and start apache, waiting a few seconds between.

browse to http:// and get https://

Sweet. 


2:15pm

### The write-up

Yup. Thought I'd be doing this five days ago, maybe four. 

Yup. Learned more than I wanted to have to.

Yup. Feeling kind of like I know a lot now. Feeling kind of confident in this domain.

I just pulled up Jupyter Lab on my phone, started a shell terminal session. I can go lots of places from there. Super sweet. Happy moments.

