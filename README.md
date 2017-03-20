# wp-admin

Shell scripts for the centralized management of WordPress. Intended to be kept as simple as possible for maximum legibility. These tools provide maintenance and workflow functions.

Used by these scripts:

* wp-cli
* apt-get
* bash
* ssh
* ubuntu
* WordPress

#### Status

Alpha. Tested on Ubuntu 16.04 with WordPress 4.7.3.

#### Goals

* Intended for use by web developers as an aid in development and to help learn terminal usage

* Backups are written constantly to facilitate easy undo

* Each script should produce machine readable output wherever relevant so scripts can be used together.


#### <a name="install"></a> Installation

Create a $5 Ubuntu 16.04 droplet. Clone this repo to your home dir, rename the folder to bin. `~/bin` is in your PATH which means you can now run the scripts from anywhere.


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

`wordpress-clone HOST` clones the live db to the staging site for that host.

```
neil@wp-admin:~$ wordpress-clone personal
Clone live db for personal to staging, search and replace urls, and flush cache.
Are you sure? y

Success: Exported to '/var/www/swap.sql'.
Success: Imported from '/var/www/swap.sql'.
+----------------------------------+-----------------------+--------------+------+
| Table                            | Column                | Replacements | Type |
+----------------------------------+-----------------------+--------------+------+
| wp_commentmeta                   | meta_key              | 0            | SQL  |
| wp_woocommerce_payment_tokenmeta | meta_value            | 0            | SQL  |
+----------------------------------+-----------------------+--------------+------+ ETC
Success: Made 155 replacements.
Staging site https://personalsite.com:2001 is now up to date with the live site https://personalsite.com
```

`wordpress-update HOST` 

```
neil@wp-admin:~$ wordpress-update personal
/tmp
Write new backup to /tmp
Rotate backups
mv: cannot stat 'personal.7.sql.gz': No such file or directory
mv: cannot stat 'personal.6.sql.gz': No such file or directory
mv: cannot stat 'personal.5.sql.gz': No such file or directory
'personal.4.sql.gz' -> 'personal.5.sql.gz'
'personal.3.sql.gz' -> 'personal.4.sql.gz'
'personal.2.sql.gz' -> 'personal.3.sql.gz'
'personal.1.sql.gz' -> 'personal.2.sql.gz'
'personal.sql.gz' -> 'personal.1.sql.gz'
Export db from WordPress...
personal.sql:    80.6% -- replaced with personal.sql.gz
Sat Mar 18 18:31:58 PDT 2017 Backup completed (tmp)
4.7.3
Success: WordPress is up to date.
Success: WordPress database already at latest db version 38590.
Success: Plugin already updated.
```

`wordpress-backup HOST MODE`  Mode is one of `manual`, `daily`, `hourly`, or `tmp` and each writes to a different directory (configured in script) No more than 8 backups are kept for each host/mode.

```
neil@wp-admin:~$ wordpress-backup personal manual
/home/neil/backups-db-manual
Write manual backup
Rotate backups
mv: cannot stat 'personal.7.sql.gz': No such file or directory
mv: cannot stat 'personal.6.sql.gz': No such file or directory
mv: cannot stat 'personal.5.sql.gz': No such file or directory
'personal.4.sql.gz' -> 'personal.5.sql.gz'
'personal.3.sql.gz' -> 'personal.4.sql.gz'
'personal.2.sql.gz' -> 'personal.3.sql.gz'
'personal.1.sql.gz' -> 'personal.2.sql.gz'
'personal.sql.gz' -> 'personal.1.sql.gz'
Export db from WordPress...
personal.sql:    80.6% -- replaced with personal.sql.gz
Sat Mar 18 18:21:14 PDT 2017 Backup completed (manual)
```

`wordpress-remote HOST "COMMAND COMMAND ..."` is for running any arbitrary wp-cli command on the remote host. Use with caution.

`wordpress-list` reads the file `~/.ssh/config` to find WordPress hosts. It should verify the connection to each and output a list of hosts that can be used in batch scripts to run `wordpress-update` or `wordpress-backup` programatically. It's a work-in-progress.






TODO:
* test for update failure
* how to roll back a failed update
* commit and push to master after updates
