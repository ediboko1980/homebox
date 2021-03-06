The files in these folders are excluded from the git repository. When deploying for the first time, they will contain
some important files, to avoid being regenerated every time you are running the Ansible scripts.

This is useful, both for archiving purposes and during the development phase. It will also alows you to restore your
server, with all the data, user accounts, certificates, passwords, encryption keys. etc. very easily.

A sub-folder named after your domain is created in this "backup" folder. So you can use the same repository to work on
multiple domains.

## Certificates

When developing and testing homebox, you can optionally backup the letsencrypt certificates. This allows you to redeploy
the certificates without requesting new ones, and therefore decrease development cycle. This is also very important, to
avoid requesting the certificates again and again, and being blocked by LetsEncrypt.

```yaml hl_lines="2"
system:
  devel: true
  debug: true
```

The certificates requested are [“staging” letsencrypt certificates](https://letsencrypt.org/docs/staging-environment/),
i.e. they do not work without displaying a warning in the client's sofwate, unless a specific certificate authority is
installed.

When using a live version, the certificates are not saved, and need to be requested again on each deployment, unless
you are using the option `system.keep_certs`:

```yaml hl_lines="4"
system:
  devel: false
  debug: false
  keep_certs: true
```

!!! Warning
    This allows you to backup live letsencrypt certificate, and can be useful in a pre-production environment, where a
    continuous integration test need real certificates. Make sure your workstation is secure enough.

See the system configuration section in the [development page](development.md#system-configuration) for more details.

For instance, here some of the certificates generated, for the domain "homebox.space"

- imap.homebox.space
- ldap.homebox.space
- smtp.homebox.space
- webmail.homebox.space
- autodiscover.homebox.space (when using autodiscover from Outlook)
- <span>www</span>.homebox.space and homebox.space (When activating the option simple web site)

When deploying the live version, the certificates are generated on the server, and not backed up.

## DKIM Keys

This folder contains the DKIM keys generated on the server. The installation script will compare these DKIM public key
with the one recorded in your DNS, and will update it only if different.

## System passwords

The ‘ldap’ folder contains the passwords generated automatically for some accounts in the LDAP directory

- admin.pwd: Super administrator password, read/write on the entire LDAP system
- manager.pwd: Manager: read/write access to user accounts
- readonly.pwd: readonly account to query the ldap server
- import.pwd: Master account used to inject imported external emails into the user's emails.
- postmaster.pwd: Postmaster account, that receives, for instance, emails like postmaster@domain.com or
  webmaster@domain.com.

## Rspamd administration interface

The Antispam comes with an excellent [web interface](https://www.rspamd.com/webui/) that provides basic functions for
setting metric actions, scores, viewing statistic and learning.

## Encryption keys

The encrypted keys generated by the system are stored in the `encryption` folder.

When you have activated the backup option, the backup encryption passphrase is stored in this folder in a file named
`backup.pwd`. __Do not loose thise file if you want to be able to restore your system from scratch__.

The file `system-key.pwd` contains a passphrase keys used to encrypt local files. This is actually used to encrypt the
credentials for the [external accounts](/external-accounts/) emails import.

## Backup Encryption keys

The backup encryption keys, generated by borg backup software are going inside the folder `backup-keys`, These keys are
named after your backup location. __Do not loose thise file if you want to be able to restore your system from
scratch__.

## SSH Keys

This folder contains the ssh keys generated for the root user, when you are using backup via an SFTP folder. You can
copy the public ssh key on the backup server, for a user named "backup". This is explained in more details in the
[backup documentation](/backup-home/).
