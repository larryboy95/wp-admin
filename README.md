# wp-admin
Status: Beta

Shell scripts for the centralized management of WordPress.
Intended to be kept as simple as possible for maximum legibility.
These tools provide maintenance and workflow functions.

This project is a dependency of [dowp](https://gitlab.com/neilscudder/dowp), deploying WordPress on Digitalocean.

Dependencies:
* Ubuntu Linux >= 16.04
* [wp-cli](https://github.com/wp-cli/wp-cli)
* [git-auto-deploy](https://github.com/olipo186/Git-Auto-Deploy)
* [restic](https://github.com/restic/restic)

For twitter notifications:
* [slack-cli](https://github.com/rockymadden/slack-cli)
* [pagres-cli](https://github.com/sindresorhus/pageres-cli)

## Installation
Clone this repo and add it to your path.
```
git clone git@gitlab.com:neilscudder/wp-admin.git
```
Add this line to your `~/.bashrc`:
```
export PATH="${PATH}:/path/to/wp-admin"
```
The `.bashrc` is loaded each time you login to Ubuntu but to load it now, run:
```
source .bashrc
```

## SSH Configuration
If you have provisioned your droplet with [dowp](https://gitlab.com/neilscudder/dowp), it will have already set up the ssh config and authorized your key, so you can skip this part.

To use these scripts and avoid entering a password every time, you must export your public ssh key to the server you're targeting.
You will also need to setup an  alias for your host as shown below. See [Digitalocean's ssh key tutorial](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-ubuntu-1804).

Host configuration in your local `~/.ssh/config`:
```
Host clientsite
    HostName 10.1.1.1
    Port 22
    User dev
    IdentityFile ~/.ssh/id_rsa
```
Enter the same info for every site you intend to maintain, with a nifty shortname (Host) for convenience and legibility. Then you can access your servers over ssh, even with custom ports, like this: `ssh clientsite` 😍

## Usage
Each script will print usage instructions if run without arguments.  
These commands each apply to a remote WordPress install.

Backup and Restore:
* [wordpress-backup](https://gitlab.com/neilscudder/wp-admin/blob/master/wordpress-backup) - Export the db and/or rsync the webroot to back them up locally. Optionally sync those backups to a restic repository.
* [wordpress-deleteAll](https://gitlab.com/neilscudder/wp-admin/blob/master/wordpress-deleteAll) - Delete all posts.
* [wordpress-permissions](https://gitlab.com/neilscudder/wp-admin/blob/master/wordpress-permissions) - Set file and folder permissions.
* [wordpress-restore](https://gitlab.com/neilscudder/wp-admin/blob/master/wordpress-restore) - Restore from local backup or git repo.

Deploying from git:
* [gad-add-host](https://gitlab.com/neilscudder/wp-admin/blob/master/gad-add-host) - Add a host to the git-auto-deploy configuration file.
* [gad-deploy-post](https://gitlab.com/neilscudder/wp-admin/blob/master/gad-deploy-post) - Runs after git-auto-deploy, rsyncs the repo from `/var/repos` to host.

Twitter Notifications:
* [notify](https://gitlab.com/neilscudder/wp-admin/blob/master/notify) - Send a tweet notification.

OS-Admin
* [os-authorize](https://gitlab.com/neilscudder/wp-admin/blob/master/os-authorize) - Authorize ssh keys for user on remote server.
* [os-update](https://gitlab.com/neilscudder/wp-admin/blob/master/os-update) - Update server software and reboot.


## Auto-Update script
The `wprdpress-update` script will use wp-cli to install updates on a remote web host.
The script then invokes `wordpress-commit` which will sync the repo and the untracked files to the local host, commit all the updates to a new branch with a unique timestamp and then push.
Extensive Slack notifications document the success or failure of automatic updates.

Example batch script for automatic updates:
```
#!/bin/bash

LOG="$HOME/auto-update.log"

notify "Auto updates are starting"

# Syntax for DO hosts using nginx:
#  wordpress-update SSH_HOST >> $LOG 2>&1
# Syntax for DO hosts using lsws:
#  wordpress-update SSH_HOST /var/www/html >> $LOG 2>&1
# Syntax for other hosts:
#  wordpress-update SSH_HOST RELATIVE_WEBROOT_PATH >> $LOG 2>&1
# (where the WEBROOT_PATH is relative to the login user's home dir)

exit 0
```
