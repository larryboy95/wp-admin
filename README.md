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

* Readability to help new users learn scripting

* Every write function must be preceeded by a backup and offer an undo

* Scripts should be single purpose and built to work as part of a higher order script

* Output should be concise and consotent for logging

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




#### <a name="usage"></a> Usage

```
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
	- Update and upgrade system software
os-install-gad HOST
	- Install and configure git-auto-deploy with the same ssh keys as the dev user
```

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

`wordpress-list` reads the file `~/.ssh/config` to find WordPress hosts. It verifies the connection to each and outputs a list of hosts that can be used in batch scripts to run `wordpress-update` or `wordpress-backup` programatically.


`os-install-gad HOST`
```
neil@wp-admin:~$ os-install-gad testing
Install/configure dev keys/git-auto-deploy on testing?
Are you sure? y
Repo url: riri
Port for git-auto-deploy [7777]:
Branch [master]:
Path [/var/www/html]:
Ssh credentials dir [~/essentials/dev-ssh]:
mkdir: cannot create directory ‘/home/dev/.ssh’: File exists
authorized_keys                                               100%  746     0.7KB/s   00:00
id_rsa                                                        100% 1679     1.6KB/s   00:00
id_rsa.pub                                                    100%  393     0.4KB/s   00:00
known_hosts                                                   100%  888     0.9KB/s   00:00
git-auto-deploy.scaffold                                      100%  333     0.3KB/s   00:00
Get:1 http://security.ubuntu.com/ubuntu xenial-security InRelease [102 kB]
Hit:2 http://ppa.launchpad.net/olipo186/git-auto-deploy/ubuntu xenial InRelease
Hit:3 http://nyc2.mirrors.digitalocean.com/ubuntu xenial InRelease
Hit:4 http://nyc2.mirrors.digitalocean.com/ubuntu xenial-updates InRelease
Hit:5 http://nyc2.mirrors.digitalocean.com/ubuntu xenial-backports InRelease
Fetched 102 kB in 0s (111 kB/s)
Reading package lists... Done
Reading package lists... Done
Building dependency tree
Reading state information... Done
software-properties-common is already the newest version (0.96.20.5).
0 upgraded, 0 newly installed, 0 to remove and 4 not upgraded.
gpg: keyring `/tmp/tmpnw1pw9oo/secring.gpg' created
gpg: keyring `/tmp/tmpnw1pw9oo/pubring.gpg' created
gpg: requesting key 0199DC32 from hkp server keyserver.ubuntu.com
gpg: /tmp/tmpnw1pw9oo/trustdb.gpg: trustdb created
gpg: key 0199DC32: public key "Launchpad PPA for Oliver Poignant" imported
gpg: Total number processed: 1
gpg:               imported: 1  (RSA: 1)
OK
Hit:1 http://mirrors.digitalocean.com/ubuntu xenial InRelease
Hit:2 http://mirrors.digitalocean.com/ubuntu xenial-updates InRelease
Hit:3 http://mirrors.digitalocean.com/ubuntu xenial-backports InRelease
Hit:4 http://ppa.launchpad.net/olipo186/git-auto-deploy/ubuntu xenial InRelease
Hit:5 http://security.ubuntu.com/ubuntu xenial-security InRelease
Reading package lists... Done
Reading package lists... Done
Building dependency tree
Reading state information... Done
git-auto-deploy is already the newest version (0.13~xenial-xenial).
0 upgraded, 0 newly installed, 0 to remove and 4 not upgraded.
Connection to testing.neilscudder.com closed.
```


