# wp-admin
Shell scripts for the centralized management of WordPress.
Part of a Digitalocean-based platform detailed in the wiki of this project.
Intended to be kept as simple as possible for maximum legibility.
These tools provide maintenance and workflow functions.

Packages and prerequisites:

* wp-cli
* apt-get
* bash
* ssh
* ubuntu
* WordPress

## Goals
* Readability for educational purposes

* Scripts should be single purpose and built to work as part of a higher order script

* Output should be concise and consistent for logging

* Each script should be portable enough for cron

## Status
Beta.
Do not run these scripts before reading them and understanding exactly what they do.
Many variables are still hard-coded to match our platform.
Slowly being developed as we add more clients to the platform.

## Installation
Create a $5 Ubuntu 16.04 droplet at Digitalocean. This droplet will serve as your administration workspace, storing all your SSH keys and database backups. Make sure it is secure.

Clone this repo to your home dir, rename the directory to `bin`. The directory `~/bin` is in your PATH which means you can now run the scripts from anywhere, like `wordpress-backup personal`

## SSH Configuration
To run these scripts remotely and avoid entering a password every time you must export your public key to the server you're targeting. [Digitalocean's ssh key tutorial](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2)

Host configuration in your local `~/.ssh/config`:

```
Host clientsite
        HostName www.clientsite.com
        Port 22
        User dev
        IdentityFile ~/.ssh/id_rsa
```
Enter the same info for every site you intend to maintain, with a nifty shortname (Host) for convenience and legibility. Then you can access your servers over ssh, even with custom ports, like this: `ssh clientsite` 😍

## Usage
```
wordpress-list
	- Checks every host in ~/.ssh/config for a compatible WP install, outputs clean list of hosts that pass the test
wordpress-backup HOST FREQUENCY
	- Where FREQUENCY is one of hourly, daily, manual or tmp. Defaults to manual.
wordpress-restore HOST FREQUENCY
	- Restores the latest backup for HOST in the FREQUENCY dir
wordpress-clone HOST
	- Clones the live db for HOST to /var/www/stagin site on same server
wordpress-update HOST
	- Updates core and plugins
wordpress-commit HOST
	- Commits and pushes changes to master branch (for post-update)
wordpress-install HOST
	- Configures web and db servers, LetsEncrypt SSL, installs WP. For use with new LEMP droplets at Digitalocean created with the cloud-config script in `assets/`
wordpress-remote HOST "CMD CMD CMD CMD"
	- Run arbitrary wp-cli commands on HOST
wordpress-deleteAll
	- Delete all posts
os-update HOST
	- Update and upgrade system software
os-install-gad HOST
	- Install and configure git-auto-deploy with the same ssh keys as the dev user
```



