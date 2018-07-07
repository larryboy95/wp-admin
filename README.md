# wp-admin
Deploy and manage WordPress on Ubuntu at Digitalocean.

Dependencies:

* Ubuntu Linux
* Php / MySql / Nginx
* LetsEncrypt
* WordPress
* wp-cli
* doctl
* git-auto-deploy
* Restic
* Ansible

## Goals
* Highly opinionated, our goal is to have a definitive answer for every question
* Scripts should be single purpose and built to work as part of a higher order script
* Readability for educational purposes
* Output should be concise and consistent for logging
* Each script should be portable enough for cron

## Status
Beta.
Do not run these scripts before reading them and understanding exactly what they do.
Many variables are still hard-coded to match this platform.
This project is in transition to a new version.
Near future plans include:
- cloning sites for installing and staging updates
- script to change domain of a running site

## Installation
Clone this repo and add it to your PATH. 
The directory `~/bin` is in your PATH which means you can now run the scripts from anywhere, like `wordpress-backup personal`

## Configuration
You will need:
- a token from a digitalocean account
- a domain using digitalocean nameservers in that account
- a private/public key pair (run `ssh-keygen -t rsa`)

`do-wp-new` will create:
- a droplet
- an entry in the ssh config file
- an entry in the ansible hosts file
- an A record for the domain

You will create one configuration file per DO account which can be used to deploy any number of droplets.

## Letsencrypt
By default the web server will use ssl but will still show a red icon.
If you have a wildcard cert for the domain (in /etc/letsencrypt/archive) then it will be used.
If you specify `WILDCARD="no"` then a certificate will be generated.
Generated certs will auto-renew.

## Usage
Each script will print usage instructions if run without arguments, i.e.:

```
$ do-wp-new

Deploys a WordPress site to a fresh droplet at digitalocean.

Requires a configuration file with Digitalocean API credentials, domain, email and ssh key info, (see below).

With the bare minimum arguments (HOSTNAME), the script will download and install the latest version of Wordpress on the new droplet.

If DB and CODEBASE are set, the script will then proceed to deploy an existing WordPress site with a fresh configuration file.

Each droplet is configured with letsencrypt. You may supply a wildcard certificate in '/etc/letsencrypt/archive/FQDN' or request a certificate for the domain during the provisioning process (WILDCARD=\"no\").

Warning: If the host already exists in SSH config the script will  overwrite the contents of the existing droplet.

[ Note: This script is not be used to deploy directly to production. The purpose here is to rapidly spin up sites that will then be used for staging or modified by hand for use as a production server. ]

---

Usage: do-wp-new CONFIGFILE HOSTNAME [DB CODEBASE]

CONFIGFILE is the path to the config file for your DO account
HOSTNAME (alphanumeric and dashes only) 

Optional args with no defaults.
Set these to restore site from backup.
    DB path to (.sql.gz) db to restore
    CODEBASE path of webroot to restore

Config file vars required, (see do-new-wp.conf.example):
    DOMAIN the root domain where you will create subdomains
    EMAIL for letsencrypt and wordpress admin user
    DOTOKEN from the API tab of Digitalocean
    PUBLICKEY fingerprint of your ssh key, added to
      the team security pane in Digitalocean

ADMINIP is automatically obtained from eth0
(ssh access is restricted to this IP)

Optional vars and their default values:
    SUBDOMAIN defaults to HOSTNAME
    SSHHOSTS ~/.ssh/config
    ANSIBLEHOSTS /etc/ansible/hosts
    PRIVATEKEY ~/.ssh/id_rsa
    DROPLETSIZE s-1vcpu-1gb
    IMAGE ubuntu-16-04-x64
    REGION sfo2
    WPUSER admin
    WILDCARD defaults to yes
    ELASTICSEARCH defaults to no
```
