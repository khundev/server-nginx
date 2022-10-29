# download the key from nginx
wget http://nginx.org/keys/nginx_signing.key

# add the key
apt-key add nginx_signing.key

# add the nginx into repository 
nano /etc/apt/sources.list.d/nginx.list
deb [arch=amd64] http://nginx.org/packages/mainline/ubuntu/ focal nginx

# then you can install nginx now
apt install nginx

# check the config file before reloading
nginx -t

# display current configuration
nginx -T

# reload the nginx configuration
nginx -s reload

# configuration files location
# main file
/etc/nginx/nginx.conf

# includes
/etc/nginx/conf.d/*.conf

