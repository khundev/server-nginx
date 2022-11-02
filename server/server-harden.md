# add ueser and password
useradd -m -s /bin/bash user-name && passwd password

# add user to group
usermod -aG group-name user-name

# show the group of user
groups user-name

# change user
su - user-name

# delete user and his directory: you will need admin privileges
sudo userdel -r user-name

# generate ssh key
ssh-keygen

# view user's ssh keys
ls -l ~/.ssh

# copy your ssh public key into the host
ssh-copy-id -i ~/.ssh/your-key.pub username@hostip

# open the file below and set PermitRootLogin no 
nano /etc/ssh/ssh_config

# add allowed uers into the ssh config file
AllowUsers user-name

# restart the ssh service
sudo systemctl restart sshd

# view inbound connections, open ports and state 
sudo ss -atpu



