# wp-admin
Bash scripts for the centralized management of WordPress running on remote servers.
Intended to be kept as simple as possible for maximum legibility.

Dependencies:
* Ubuntu Linux >= 16.04
* [notify-cli](https://gitlab.com/neilscudder/notify-cli)
* [wp-cli](https://github.com/wp-cli/wp-cli)

## Installation
Clone this repo and add it to your path.
```
git clone git@gitlab.com:neilscudder/wp-admin.git /path/to/wp-admin
```

Add this line to your `~/.bashrc`:
```
export PATH="${PATH}:/path/to/wp-admin"
```

The `.bashrc` is loaded each time you login to Ubuntu but to load it now, run:
```
source ~/.bashrc
```

## SSH Configuration
To use these scripts and avoid entering a password every time, you must export your public ssh key to the server you're targeting.
You will also need to setup an  alias for your host as shown below. See [Digitalocean's ssh key tutorial](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-ubuntu-1804).

Host configuration in your local `~/.ssh/config`:
```
Host clientsite
    HostName 10.1.1.1
    Port 22
    User neil
    IdentityFile ~/.ssh/id_rsa
```
Enter the same info for every site you intend to maintain, with a nifty shortname (Host) for convenience and legibility. Then you can access your servers over ssh, even with custom ports, like this: `ssh clientsite` 😍

## Usage
Each script will print usage instructions if run without arguments.  
