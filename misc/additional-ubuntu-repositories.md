# Additional Ubuntu Repositories

Depending on your Ubuntu installation, you may need this to use the "add-apt-repository" command:

	apt install software-properties-common

Certbot (Let's Encrypt)
	
	add-apt-repository ppa:certbot/certbot

	apt update && apt install -y  certbot

Geth (Go Ethereum)

	add-apt-repository ppa:ethereum/ethereum

	apt update && apt install -y ethereum

Sublime-Text

	wget -qO - https://download.sublimetext.com/sublimehq-pub.gpg | sudo apt-key add -

	apt install apt-transport-https
	
	echo "deb https://download.sublimetext.com/ apt/stable/" | sudo tee /etc/apt/sources.list.d/sublime-text.list
	
	apt update && apt install -y sublime-text
