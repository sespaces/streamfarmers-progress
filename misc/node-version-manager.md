# a gist for using NVM instead of NPM

Set version numbers to recent and proven

	$ export NVM_VERSION=0.34.0

	$ export NPM_VERSION=10.15.3


Install Node Version Manager

	$ curl -o- "https://raw.githubusercontent.com/creationix/nvm/v$NVM_VERSION/install.sh" | bash


Activate NVM

	$ export NVM_DIR="$HOME/.nvm"
	
	$ [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" 
	
	$ [ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion" 



Install Node Package Manager

	$ nvm install $NPM_VERSION 