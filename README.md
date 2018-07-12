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

## Quickstart

`do-wp-new account.conf hostname backup.sql.gz backups/site/html`

This command will:
* Check if your ssh key is registered to the DO account and add it if necessary
* Add the host to your ssh config file
* Add the host to your ansible hosts file
* If hostname is found you can restore to it right away
* Create a droplet at Digitalocean (DO)
* Create DNS records for hostname at the domain supplied in the config file, including a cname for the www variant
* Install nginx, php and mysql configured for WordPress hosting
* Install certbot and elasticsearch (optional)
* Copy a wildcard certificate and configure SSL (see [le-wildcard](le-wildcard))
* Create user 'dev' with sudo priveleges
* Restrict ssh access to the IP of the deploying machine
* Download and install the latest version of WordPress
* Deploy an existing site from backup
* Search and replace the site URL
* Run [wordpress-permissions](wordpress-permissions) to chown the webrrot for user 'dev'
* Run [os-authorize](os-authorize) to add authorized ssh keys for user 'dev'


## How to Migrate a Site

Prep: 
* Setup `oldserver` in your ssh config file
* Create a domain in the Digitalocean control panel for your client like this: do.clientsite.com
* Login to the client's DNS provider and create NS records for the subdomain do to the three Digitalocean nameservers.
* Create an API token in the client's DO control panel and copy it to your clipboard
* Run `le-wildcard` and give it your token
* Create `account.conf` with at least the required paramters, (see [do-wp-new.sample.conf](do-wp-new.sample.conf))

The following command will write to `/var/backups/wp-db-backups` and `/var/backups/wp-full-backups` (by default):

```
$ wordpress-backup oldserver full /path/to/webroot
```

This command will deploy that backup to `https://hostname.do.clientsite.com`:

```
$ do-wp-new account.conf HOSTNAME /var/backups/wordpress/oldserver.sql.gz /var/backups/wordpress/oldserver/html
```

## W-I-P Notes
The provisioning script works best at the moment if you have a wildcard certificate for the domain you are provisioning to.
The le-wildcard script will do this for you in one step as long as you have a digitalocean token.

The www prefix is automatically redirected to the bare domain.

## Installation
Clone this repo and add it to your PATH, or just put it in `~/bin` 

## Configuration
You will need:
- a token from a digitalocean account
- a domain using digitalocean nameservers in that account
- a private/public key pair (run `ssh-keygen -t rsa`)

You must create a configuration file and a wildcard certificate per DO account which can be used to deploy any number of droplets.

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

Requires a configuration file with Digitalocean API credentials, domain, email 
and ssh key info, (see below).

With the bare minimum arguments (HOSTNAME), the script will download and install 
the latest version of Wordpress on the new droplet.

If DB and CODEBASE are set, the script will then proceed to deploy an existing 
WordPress site with a fresh configuration file.

Each droplet is configured with letsencrypt. You may supply a wildcard 
certificate in '/etc/letsencrypt/archive/FQDN' or request a certificate for the 
domain during the provisioning process (WILDCARD=\"no\").

Warning: If the host already exists in SSH config the script will  overwrite the 
contents of the existing droplet.

[ Note: This script is not be used to deploy directly to production. The purpose 
here is to rapidly spin up sites that will then be used for staging or modified 
by hand for use as a production server. ]

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
