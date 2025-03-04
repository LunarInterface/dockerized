__WARNING__: THIS IS A BETA EDITION AND NOT ALWAYS STABLE, DO NOT TRY IT IN PRODUCTION (YET).

- Source code is hosted on [GitHub](https://github.com/iredmail/dockerized).
  Bug report, feedback, patches are always welcome.
- Base image is [Ubuntu 22.04 (jammy)](https://hub.docker.com/_/ubuntu).
- Dockerized iRedMail follows the [Best Practice of iRedMail Easy platform](https://docs.iredmail.org/iredmail-easy.best.practice.html).
- 2 tags are available:
  - `iredmail/mariadb:stable`: Stable version.
  - `iredmail/mariadb:nightly`: Triggered by __EACH__ GitHub commit.

# Quick start

Create a docker environment file used to store custom settings:

```
mkdir /iredmail         # Create a new directory or use any directory
                        # you prefer. `/iredmail/` is just an example
cd /iredmail
touch iredmail-docker.conf

echo HOSTNAME=mail.mydomain.com >> iredmail-docker.conf
echo FIRST_MAIL_DOMAIN=mydomain.com >> iredmail-docker.conf
echo FIRST_MAIL_DOMAIN_ADMIN_PASSWORD=my-secret-password >> iredmail-docker.conf
echo MLMMJADMIN_API_TOKEN=$(openssl rand -base64 32) >> iredmail-docker.conf
echo ROUNDCUBE_DES_KEY=$(openssl rand -base64 24) >> iredmail-docker.conf
```

Create required directories to store application data:

```
cd /iredmail
mkdir -p data/{backup-mysql,clamav,custom,imapsieve_copy,mailboxes,mlmmj,mlmmj-archive,mysql,sa_rules,ssl,postfix_queue}
```

Launch the container:

```
docker run \
    --rm \
    --name iredmail \
    --env-file iredmail-docker.conf \
    --hostname mail.mydomain.com \
    -p 80:80 \
    -p 443:443 \
    -p 110:110 \
    -p 995:995 \
    -p 143:143 \
    -p 993:993 \
    -p 25:25 \
    -p 465:465 \
    -p 587:587 \
    -v /iredmail/data/backup-mysql:/var/vmail/backup/mysql \
    -v /iredmail/data/mailboxes:/var/vmail/vmail1 \
    -v /iredmail/data/mlmmj:/var/vmail/mlmmj \
    -v /iredmail/data/mlmmj-archive:/var/vmail/mlmmj-archive \
    -v /iredmail/data/imapsieve_copy:/var/vmail/imapsieve_copy \
    -v /iredmail/data/custom:/opt/iredmail/custom \
    -v /iredmail/data/ssl:/opt/iredmail/ssl \
    -v /iredmail/data/mysql:/var/lib/mysql \
    -v /iredmail/data/clamav:/var/lib/clamav \
    -v /iredmail/data/sa_rules:/var/lib/spamassassin \
    -v /iredmail/data/postfix_queue:/var/spool/postfix \
    iredmail/mariadb:stable
```

Notes:

- On first run, it will generate a self-signed ssl cert, this may take a long
time, please be patient.
- Each time you run the container, few tasks will be ran:
    - Update SpamAssassin rules.
    - Update ClamAV virus signature database.
- `FIRST_MAIL_DOMAIN_ADMIN_PASSWORD` is only set/reset on first run, not each run.
- All SQL passwords are randomly set/reset by default each time you launch or
  relaunch the container. If you don't like this, please set fixed passwords
  in `iredmail-docker.conf`, e.g. `MYSQL_ROOT_PASSWORD=<your-password>`.
- Do not forget to [setup DNS records](https://docs.iredmail.org/setup.dns.html)
  for your server hostname and email domain names.
- If you're running Docker on Windows and macOS, container will fail to launch
  and you must switch to docker volumes as described below.

If you're running Docker on Windows and macOS, or you just prefer storing
persistent data in Docker volumes, please create required volumes:

```
docker volume create iredmail_backup           # Backup copies
docker volume create iredmail_mailboxes        # All users' mailboxes
docker volume create iredmail_mlmmj            # mailing list data
docker volume create iredmail_mlmmj_archive    # mailing list archive
docker volume create iredmail_imapsieve_copy   # Used by Dovecot plugin 'imapsieve'
docker volume create iredmail_custom           # custom config files
docker volume create iredmail_ssl              # SSL cert/key files
docker volume create iredmail_mysql            # MySQL databases
docker volume create iredmail_clamav           # ClamAV database
docker volume create iredmail_sa_rules         # SpamAssassin rules
docker volume create iredmail_postfix_queue    # Postfix queues
```

Then launch the container with volumes:

```
docker run \
    --rm \
    --name iredmail \
    --env-file iredmail-docker.conf \
    --hostname mail.mydomain.com \
    -p 80:80 \
    -p 443:443 \
    -p 110:110 \
    -p 995:995 \
    -p 143:143 \
    -p 993:993 \
    -p 25:25 \
    -p 465:465 \
    -p 587:587 \
    -v iredmail_backup-mysql:/var/vmail/backup/mysql \
    -v iredmail_mailboxes:/var/vmail/vmail1 \
    -v iredmail_mlmmj:/var/vmail/mlmmj \
    -v iredmail_mlmmj_archive:/var/vmail/mlmmj-archive \
    -v iredmail_imapsieve_copy:/var/vmail/imapsieve_copy \
    -v iredmail_custom:/opt/iredmail/custom \
    -v iredmail_ssl:/opt/iredmail/ssl \
    -v iredmail_mysql:/var/lib/mysql \
    -v iredmail_clamav:/var/lib/clamav \
    -v iredmail_sa_rules:/var/lib/spamassassin \
    -v iredmail_postfix_queue:/var/spool/postfix \
    iredmail/mariadb:stable
```


# Use Docker-Compose
Docker Compose build file provided to `docker-compose.yml`.

This docker compose file use volume bind, insted of docker volume, Unlike the docker run example above.

The reason is to access the `.data/ directory` path in a file format to __restore, recover, and expand the volume bind__ to high availability in __docker-compose, Kubernetes__.

Notes:

- carefully config `docker-compose.yml` file, especially docker __Bind Volume Path__.
- __Do not modify__ `.data/ directory`, and __Access__, or __Remove__.
- Must __backup__ your `.data/ directory`.

Compose build with logs:
```
docker-compose build
```

Compose build with no cache:
```
docker-compose build --no-cache
```

And Compose Start with logs:
```
# Compose up with logs. (Ctrl + C, or exit means contaier stop.)
docker-compose up

# Compose up with no logs. (Deattached, Even containers run backround.)
docker-compose up -d
```

Use to `docker-compose logs` or `docker-compose exec`.
```
# Compose display logs.
docker-compose logs

# Compose attach bash with input output tty. (attach, Even containers run and, it use to EXEC(unix process method) bash process.)
docker-compose exec -it iredmail bash
```

Finalley, You possible debug, or Supervisor Status check in the IRedmain Container.
```
# View raw log.
ls -alh /var/log/

# Supervisor damon display sub process status.
supervisorctl status
```



# Overview

Only one config file `iredmail-docker.conf` on Docker host.

This file is optional if you prefer overwriting parameters with the `-e`
argument while launching container. For example:

```
docker run -e HOSTNAME=mail.mydomain.com -e FIRST_MAIL_DOMAIN=mydomain.com ...
```

We recommend storing them in an env file (`iredmail-docker.conf` in our
example) to save some typing each time you launch the container.

# Required parameters

There're few __REQUIRED__ parameters you __MUST__ set in `iredmail-docker.conf`:

```
# Server hostname. Must be a FQDN. For example, mail.mydomain.com
HOSTNAME=

# First mail domain name. For example, mydomain.com.
FIRST_MAIL_DOMAIN=

# (Plain) password of mail user `postmaster@<FIRST_MAIL_DOMAIN>`.
FIRST_MAIL_DOMAIN_ADMIN_PASSWORD=

# A secret token used for accessing mlmmjadmin API.
MLMMJADMIN_API_TOKEN=

# The secret string used to encrypt/decrypt Roundcube session data.
# Required if you need to run Roundcube webmail.
# You can generate random string with command `openssl rand -base64 24` as the
# des key.
# Every time this key changed, all Roundcube session data becomes invalid and
# users will be forced to re-login.
ROUNDCUBE_DES_KEY=
```

Notes:

- `iredmail-docker.conf` will be read by Docker as an environment file,
  any single quote or double quote will be treated as part of the value.
  __Do not use any whitespace, tab in value, and no single or double quotes.__
- It will be imported as bash shell script too.

There're many OPTIONAL settings defined in file
`/docker/entrypoints/settings.conf` inside docker container,
you'd like to change any of them, please write the same parameter name with
your custom value in `iredmail-docker.conf` to override it.

# Optional parameters

```
# Define your custom https port if you don't want to the default one (443).
PORT_HTTPS=4443
```

# Hardware requirements

- At least 4GB RAM is required for a low traffic production mail server.

# Installed softwares

- Postfix: SMTP server.
- Dovecot: POP3/IMAP/LMTP/Sieve server, also offers SASL AUTH service for Postfix.
- mlmmj: mailing list manager.
- mlmmjadmin: RESTful API server used to manage (mlmmj) mailing lists.
- Amavisd-new + ClamAV + SpamAssassin: anti-spam and anti-virus, DKIM signing and verification, etc.
- iRedAPD: Postfix policy server. Developed by iRedMail team.
- Fail2ban: scans log files and bans bad clients.
- Roundcube: webmail.
- iRedAdmin: web-based admin panel, open source edition.

You may want to check [this tutorial](https://docs.iredmail.org/network.ports.html)
to figure out the mapping of softwares and network ports.

# Exposed network ports

- 80: HTTP
- 443: HTTPS
- 25: SMTP
- 465: SMTPS (SMTP over SSL)
- 587: SUBMISSION (SMTP over TLS)
- 143: IMAP over TLS
- 993: IMAP over SSL
- 110: POP3 over TLS
- 995: POP3 over SSL
- 4190: Managesieve service

# Links

- If you're trying to deploy this docker image with Ansible, user @ricristian
  already have some code for your reference: <https://github.com/ricristian/ansible-dockerized-iredmail>
