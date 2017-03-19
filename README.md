# wp-admin

Shell scripts for the centralized management of WordPress. Intended to be kept as simple as possible for maximum legibility. These tools provide maintenance and workflow functions.

* wp-cli
* apt-get
* bash
* ssh
* ubuntu
* WordPress

#### Status

Proof-of-concept. The text below is being adapted from another article.


#### <a name="install"></a> Installation

Clone to your home dir, rename the folder to bin. `~/bin` is in your PATH so you can now run from anywhere.


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


#### <a name="updates"></a> Updates

You can call the update script below using that same shortname: `wordpress-update clientsite` and pass it straight through to SSH (it becomes the variable $1):
```
#!/bin/bash
args="--path=/var/www/html --ssh=$1"
wp $args core version
wp $args core update
wp $args core update-db
wp $args plugin update --all
exit 0
```

This is how it runs:

```
neil@wp-admin:~$ wordpress-update clientsite
4.7.3
Success: WordPress is up to date.
Success: WordPress database already at latest db version 38590.
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

TODO:
* test for update failure
* how to roll back a failed update
* commit and push to master after updates


#### <a name="clone"></a> Clone Database

To copy the live database to the staging environment and update all url's with the staging site port, there is a simple script that uses WP-CLI. 


```
dev@web:~/bin$ clonedb clientsite
Clone live db to staging, search and replace urls, and flush cache.
Are you sure? y
Success: Exported to '/var/www/swap.sql'.
Success: Imported from '/var/www/swap.sql'.
[mess of sql output]
Success: Made 147 replacements.
Success: The cache was flushed.
https://dev.clientsite.com:2001 is now up to date with the live db.
dev@web:~/bin$
```

You now have an exact and current copy of the live site running on the same web server as the live site, providing the most accurate testing environment possible.


#### <a name="backups"></a> Backups

```
neil@wp-admin:~$ wordpress-backup clientsite
Rotate backups
'clientsite.4.sql.gz' -> 'clientsite.5.sql.gz'
'clientsite.3.sql.gz' -> 'clientsite.4.sql.gz'
'clientsite.2.sql.gz' -> 'clientsite.3.sql.gz'
'clientsite.1.sql.gz' -> 'clientsite.2.sql.gz'
'clientsite.sql.gz' -> 'clientsite.1.sql.gz'
Export db from WordPress...
clientsite.sql:        90.1% -- replaced with clientsite.sql.gz
neil@wp-admin:~$
```

TODO add section on restoring backups, downloading backups

