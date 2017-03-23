# wp-admin

Shell scripts for the centralized management of WordPress. Intended to be kept as simple as possible for maximum legibility. These tools provide maintenance and workflow functions.

Packages and prerequisites:

* wp-cli
* apt-get
* bash
* ssh
* ubuntu
* WordPress

#### Status

Beta. Tested on Ubuntu 16.04 with WordPress 4.7.3.

#### Goals

* Intended for use by web developers as an aid in development and to help learn terminal server administration

* Every write function must be preceeded by a backup and offer an undo

* Each script should produce machine readable output wherever relevant so scripts can be used together

* Each script should be portable enough for cron


#### <a name="install"></a> Installation

Create a $5 Ubuntu 16.04 droplet at Digitalocean. This droplet will serve as your administration workspace, storing all your SSH keys and database backups. Make sure it is secure.

Clone this repo to your home dir, rename the directory to `bin`. The directory `~/bin` is in your PATH which means you can now run the scripts from anywhere, like `wordpress-backup personal`


#### <a name="ssh"></a> SSH Configuration

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


#### <a name="how"></a> How It Works

You call each script below using that same Host: `wordpress-update clientsite` and pass it straight through to SSH (it becomes the variable $1). For example:
```
#!/bin/bash
wp --path=/var/www/html --ssh=$1 plugin update --all
```

This is how it runs:

```
neil@wp-admin:~$ wordpress-update clientsite
Enabling Maintenance mode...
Downloading update from https://downloads.wordpress.org/plugin/woocommerce-gateway-stripe.3.1.1.zip...
Unpacking the update...
Installing the latest version...
Removing the old version of the plugin...
Plugin updated successfully.
Disabling Maintenance mode...
+----------------------------+-------------+-------------+---------+
| name                       | old_version | new_version | status  |
+----------------------------+-------------+-------------+---------+
| woocommerce-gateway-stripe | 3.0.7       | 3.1.1       | Updated |
+----------------------------+-------------+-------------+---------+
Success: Updated 1 of 1 plugins.
neil@wp-admin:~$
```



#### <a name="usage"></a> Usage

wordpress-list
	- Checks every host in ~/.ssh/config for a compatible WP install, outputs clean list of hosts that pass the test
wordpress-backup HOST FREQUENCY
	- Where FREQUENCY is one of hourly, daily, manual or tmp
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
	- Basic apt-get stuff


#### <a name="examples"></a> Examples

`wordpress-clone HOST` clones the live db to the staging site for that host.

```
neil@wp-admin:~$ wordpress-clone clientsite
Write new backup to /tmp
Rotate clientsite backups
Export db from WordPress...
clientsite.sql:        90.1% -- replaced with clientsite.sql.gz
Tue Mar 21 10:58:02 PDT 2017 Backup completed for clientsite (tmp)
Clone live db for clientsite to staging, search and replace urls, and flush cache.
Are you sure? y
Success: Exported to '/var/www/swap.sql'.
Success: Imported from '/var/www/swap.sql'.
+----------------------------------+-----------------------+--------------+------+
| Table                            | Column                | Replacements | Type |
+----------------------------------+-----------------------+--------------+------+
| wp_commentmeta                   | meta_key              | 0            | SQL  |
| wp_commentmeta                   | meta_value            | 0            | SQL  |
| wp_comments                      | comment_author        | 0            | SQL  |
| wp_comments                      | comment_author_email  | 40           | SQL  | 0            | SQL  |
| wp_woocommerce_payment_tokenmeta | meta_value            | 0            | SQL  |
+----------------------------------+-----------------------+--------------+------+
Success: Made 155 replacements.
Success: The cache was flushed.
Staging site https://clientsite.com:2001 is now up to date with the live site https://clientsite.com
```

`wordpress-update HOST` 

```
neil@wp-admin:~/bin$ wordpress-update personal
Write new backup to /tmp
Rotate personal backups
Export db from WordPress...
personal.sql:    75.1% -- replaced with personal.sql.gz
Tue Mar 21 16:48:28 PDT 2017 Backup completed for personal (tmp)
4.7.3
Success: WordPress is up to date.
Success: WordPress database already at latest db version 38590.
Success: Plugin already updated.
Connection to personal.neilscudder.com closed.
```

`wordpress-backup HOST MODE`  Mode is one of `manual`, `daily`, `hourly`, or `tmp` and each writes to a different directory (configured in script) No more than 8 backups are kept for each host/mode.

```
neil@wp-admin:~/bin$ wordpress-backup personal tmp
Export db from WordPress...
Success: Exported to '/var/www/personal.sql'.
/var/www/personal.sql:   75.0% -- replaced with /var/www/personal.sql.gz
personal.sql.gz                                               100%  127KB 126.6KB/s   00:01
Tue Mar 21 22:06:03 PDT 2017 Backup completed for personal (tmp)
```

`wordpress-remote HOST "COMMAND COMMAND ..."` is for running any arbitrary wp-cli command on the remote host. Use with caution.

`wordpress-list` reads the file `~/.ssh/config` to find WordPress hosts. It should verify the connection to each and output a list of hosts that can be used in batch scripts to run `wordpress-update` or `wordpress-backup` programatically. It's a work-in-progress.






TODO:
* test for update failure
* how to roll back a failed update
* commit and push to master after updates
