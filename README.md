# mbsync-dovecot
This is an example docker compose file for the mbsync and dovecot containers. These containers were created for archival purposes of a remote email server. Email is synced locally in Maildir format. Deletions on remote server are not propagated. Dovecot then acts as an imap server. I use mine locally since my devices have split tunnel wireguard. I suppose you could expose your email server to the internet but that would scare me.

Once you make the mbsync container, you need a cron job or something so that it'll autorun every X minutes (I have mine run every 5 minutes)

## Dovecot
| Environment Variable | Description |
| --- | --- |
| email_username | local username |
| email_password | local password |
| PUID | UID, same as mbsync PUID |
| disable_plaintext_auth | yes or no |
| ssl_cert | filename, ex. fullchain1.pem |
| ssl_key | filename, ex. privkey1.pem |
| ssl | required, yes, or no |
| server_address | local imap server address |

## Mbsync
| Environment Variable | Description |
| --- | --- |
| host_name | remote imap server address |
| port | remote imap server port |
| remote_username | remote username |
| remote_password | remote password |
| ssltype | None,STARTTLS, or IMAPS |
| sslversions | SSLv3, TLSv1, TLSv1.1, TLSv1.2, TLSv1.3 |
| mailbox_name | any name you want for the mailbox (ex. gmail) |
| PUID | UID, same as dovecot PUID |

Dovecot Repo - https://github.com/jon6fingrs/dovecot
Mbsync Repo - https://github.com/jon6fingrs/mbsync

Dovecot Docker Hub - https://hub.docker.com/r/thehelpfulidiot/dovecot
Mbsync Docker Hub - https://hub.docker.com/r/thehelpfulidiot/mbsync
