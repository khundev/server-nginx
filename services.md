# list the services that are running
systemctl list-units --type

# look at specific unit that you want to view in journal
journalctl -u apache2
journalctl -u ssh

# watch live service
journalctl -u ssh -f

# watch since particular time
journalctl -u ssh --since "2022-10-28 9:00:00"

