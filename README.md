# wp-admin
Version 0.1.3
Status: Beta

Shell scripts for the centralized management of WordPress.
Part of a Digitalocean-based platform detailed in the wiki of this project.
Intended to be kept as simple as possible for maximum legibility.
These tools provide maintenance and workflow functions.

Dependencies:

* Ubuntu Linux
* WordPress
* wp-cli
* git-auto-deploy
* Restic

## Goals
* Highly opinionated, our goal is to have a definitive answer for every question
* Scripts should be single purpose and built to work as part of a higher order script
* Readability for educational purposes
* Output should be concise and consistent for logging

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
Each script will print usage instructions if run without arguments.

