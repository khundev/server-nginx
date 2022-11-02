## change password
alter user db-user-name password 'your-password';

## command to review the config file location
show config_files;

## view extracted config file
cat /etc/postgresql/9.3/main/postgresql.conf

## view hba conf file 
cat /etc/postgresql/9.3/main/pg_hba.conf

## reloading the configuration settings
pg_ctl reload

# ssl_certificate location in db server
ssl_cert_file = '/etc/ssl/certs/ssl-cert-snakeoil.pem'		# (change requires restart)
ssl_key_file = '/etc/ssl/private/ssl-cert-snakeoil.key'		# (change requires restart)

