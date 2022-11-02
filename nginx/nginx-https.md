# see the os release
cat /etc/os-release

# install certbot and its dependencies for certification
apt install certbot python-certbot-apache
apt install certbot python-certbot-nginx

# run this certbot command
certbot --apache -d your-domain
certbot --nginx -d your-domain

