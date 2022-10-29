# install unusual firewall
apt install ufw

# view, enable, and start the service
systemctl status ufw
systemctl enable ufw
systemctl start ufw

# default outbound allow to internet
ufw default allow outgoing

# default inbound deny 
ufw default deny incoming

# allow ssh connection
ufw allow ssh

# allow http connection 
ufw allow http/tcp

# allow your localhost to connect to the server
ufw allow from yourhostip to any port 22 proto tcp

# check your firewall rules
ufw status

# delete rule
ufw delete rule-number

