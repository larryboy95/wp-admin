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
* Highly opinionated, our goal is to have a definitive answer for every question

* Scripts should be single purpose and built to work as part of a higher order script

* Readability for educational purposes

* Output should be concise and consistent for logging

* Each script should be portable enough for cron

## Status
Beta.
Do not run these scripts before reading them and understanding exactly what they do.
Many variables are still hard-coded to match this platform.

## Installation
Create a $5 Ubuntu 16.04 droplet at Digitalocean. This droplet will serve as your administration workspace, storing all your SSH keys and database backups. 
Make sure it is secure.

Clone this repo to your home dir, rename the directory to `bin`. 
The directory `~/bin` is in your PATH which means you can now run the scripts from anywhere, like `wordpress-backup personal`

## SSH Configuration
To run these scripts remotely and avoid entering a password every time you must export your public key to the server you're targeting. 
[Digitalocean's ssh key tutorial](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2.)

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

- wordpress-list
	- Checks every host in ~/.ssh/config for a compatible WP install, outputs clean list of hosts that pass the test
- wordpress-backup HOST FREQUENCY
	- Where FREQUENCY is one of hourly, daily, manual or tmp. Defaults to manual
- wordpress-backup-all FREQUENCY
	- Script in alpha status, use with caution
	- Reads a file ~/wp.sitelist with one Host alias per line
	- Frequency argument is not optional
	- Use `wordpress-list > ~/wp.sitelist` to update site list
	- Executes the wordpress-backup script once per host in parallel
- wordpress-restore HOST FREQUENCY
	- Script in alpha status, use with caution
	- Restores the latest backup for HOST in the FREQUENCY dir
- wordpress-update HOST
	- Updates core only
- wordpress-update-plugins HOST
	- Updates all plugins (use with caution)
- wordpress-permissions HOST USER
	- Will prompt for user name if not supplied
	- Used in the update script to switch ownership back and forth between dev and git-auto-deploy
- wordpress-commit HOST
	- Commits and pushes changes (for post-update)
- wordpress-install HOST
	- Configures web and db servers, LetsEncrypt SSL, installs WP. 
	- For use with new LEMP droplets at Digitalocean created with the cloud-config script in `assets/`
- wordpress-remote HOST "CMD CMD CMD CMD"
	- Run arbitrary wp-cli commands on HOST
- wordpress-deleteAll
	- Delete all posts
- os-update HOST
	- Update and upgrade system software
- os-setup-gad HOST
	- Install and configure git-auto-deploy with the same ssh keys as the dev user
- os-setup-certbot HOST
	- Install and configure automatic SSL certificate updates
	- Requires fully configured server with LetsEncrypt certs already installed



