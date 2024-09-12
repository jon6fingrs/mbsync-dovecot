# mbsync-dovecot
This is an example docker compose file for the mbsync, dovecot, and now solr containers. I did not write nor I am affiliated with the authors of these applications. These containers were created for archival purposes of a remote email server. Email is synced locally in Maildir format. Deletions on remote server are not propagated. Dovecot then acts as an imap server. Solr indexes the mailbox and facilitates searches.
I use mine locally since my devices have split tunnel wireguard. I suppose you could expose your email server to the internet but that would scare me.

Previously, the mbsync container would run, complete the sync, and quit. It required a cron job or some other task scheduler on the host system to update the local mailbox every so often. That is still an option, however, there is now a "cron" environment variable. if it is set to yes, after mbsync finishes its task, cron will open as the primary process in the docket container and call mbsync every 5 minutes. 

## Dovecot
| Environment Variable | Description |
| --- | --- |
| email_username | local username to access local server |
| email_password | local password to access local server|
| PUID | UID, same as mbsync PUID, based on host system user |
| disable_plaintext_auth | yes or no (default no if omitted) |
| ssl_cert | filename, ex. fullchain1.pem |
| ssl_key | filename, ex. privkey1.pem |
| ssl | required, yes, or no (default no if omitted) |
| server_address | local imap hostname for this server |
| df_pem | will initialize and enable dh key exchange |

## Mbsync
| Environment Variable | Description |
| --- | --- |
| host_name | remote imap server address |
| port | remote imap server port |
| remote_username | remote username |
| remote_password | remote password |
| ssltype | None,STARTTLS, or IMAPS (default IMAPS) |
| sslversions | SSLv3, TLSv1, TLSv1.1, TLSv1.2, TLSv1.3 (default TLSv1) |
| mailbox_name | any name you want for the mailbox (ex. gmail) |
| PUID | UID, same as dovecot PUID |
| cron | yes or no (default no, if enabled, will keep container active and run email sync every 5 minites)

## Solr
No real environment variables to configure. 

## Automatically Fetch Emails (if mbsync cron environment variable is off)

This container needs either a cron job or systemd timer for the following command:

docker container start <container name>

I run this on my synology and have that command running every 5 minutes through the system tasks control panel.
  
Here is an example on how to call the container automatically every 5 minutes taken from the ArchWiki mbsync page. Obviously, the article has the example for the normal repo version of the application and so below is modified and should work for the docker version but please let me know if not.
  
https://wiki.archlinux.org/title/isync#Calling_mbsync_automatically
  

```
~/.config/systemd/user/mbsync.service

[Unit]
Description=Mailbox synchronization service

[Service]
Type=oneshot
ExecStart=docker container start <container name>

[Install]
WantedBy=default.target
```
  
The following timer configures mbsync to be started 2 minutes after boot, and then every 5 minutes:
```
~/.config/systemd/user/mbsync.timer

[Unit]
Description=Mailbox synchronization timer

[Timer]
OnBootSec=2m
OnUnitActiveSec=5m
Unit=mbsync.service

[Install]
WantedBy=timers.target
```
  
Once those two files are created, reload systemd, then enable and start mbsync.timer, adding the --user flag to systemctl.

Also consider using the below command to enable systemd timers for a particular user prior to login.

```
loginctl enable-linger username
```
  
From what I read, cronjobs are not ideal for a task like this which could potentially run for an extended period of time.

## Links
  
Dovecot Repo - https://github.com/jon6fingrs/dovecot
Mbsync Repo - https://github.com/jon6fingrs/mbsync
Solr Repo - https://github.com/jon6fingrs/solr

Dovecot Docker Hub - https://hub.docker.com/r/thehelpfulidiot/dovecot
Mbsync Docker Hub - https://hub.docker.com/r/thehelpfulidiot/mbsync

For bonus points, you can install a web frontend email client to access these local emails. I have an example of one for Roundcube on my blog post which inspired this project.

https://thehelpfulidiot.com/making-an-automatic-email-backup-part-2
