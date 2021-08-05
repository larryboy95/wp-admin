# Wordpress-Deploy

Deployment is handled by the `wordpress-deploy` script which is called by a git hook which resides in a git remote on an internal administration server.

Normally, code is pushed to a single `origin` remote (i.e. gitlab.)
The `origin` remote then triggers deployment using either a webhook or a built-in CI function.

To use `wordpress-deploy` we are going to  set specific remotes for branches which have servers associated with them and associate two `push` URLs per deployment-enabled remote.

`wordpress-deploy` is specifically meant to centralize deployment from a single admin server in order to simplify the management of multiple sites.

Each remote on the admin server is associated with an individual branch and a single deployment target.
The admin server configuration is handled by the (https://gitlab.com/neilscudder/script-generator)[script-generator].
More on that below.

Assumptions:
- the `origin` remote is at gitlab
- you have already cloned the repo and changed to the local repo directory
- the `main` branch gets deployed to the production server
- the `staging` branch deploys to a staging server

The `origin` remote will remain unadulterated so that new branches only get pushed to gitlab.
Two new remotes will be created, and then updated so that each has one URL to `fetch` from and two URLs to `push` to.

```
git remote add production git@gitlab.com:agency/git-deploy-test.git;
git remote set-url --push --add production agency-admin:scratch/agency/repos/git-deploy-test.agency.com.git;
git remote set-url --push --add production git@gitlab.com:agency/git-deploy-test.git;
git checkout main
git push -u production

git remote add staging git@gitlab.com:agency/git-deploy-test.git;
git remote set-url --push --add staging agency-admin:scratch/agency/repos/git-deploy-test-staging.agency.com.git;
git remote set-url --push --add staging git@gitlab.com:agency/git-deploy-test.git;
git checkout staging
git push -u staging
```

The output of `git remote -v` should now look like:
```
origin	git@gitlab.com:agency/git-deploy-test.git (fetch)
origin	git@gitlab.com:agency/git-deploy-test.git (push)
production	git@gitlab.com:agency/git-deploy-test.git (fetch)
production	agency-admin:scratch/agency/repos/git-deploy-test.agency.com.git (push)
production	git@gitlab.com:agency/git-deploy-test.git (push)
staging	git@gitlab.com:agency/git-deploy-test.git (fetch)
staging	agency-admin:scratch/agency/repos/git-deploy-test-staging.agency.com.git (push)
staging	git@gitlab.com:agency/git-deploy-test.git (push)
```

And your `.git/config` file should include a section that looks like this:
```
[remote "origin"]
	url = git@gitlab.com:agency/git-deploy-test.git
	fetch = +refs/heads/*:refs/remotes/origin/*
[branch "main"]
	remote = production
	merge = refs/heads/main
[remote "production"]
	url = git@gitlab.com:agency/git-deploy-test.git
	fetch = +refs/heads/*:refs/remotes/production/*
	pushurl = agency-admin:scratch/agency/repos/git-deploy-test.agency.com.git
	pushurl = git@gitlab.com:agency/git-deploy-test.git
[remote "staging"]
	url = git@gitlab.com:agency/git-deploy-test.git
	fetch = +refs/heads/*:refs/remotes/staging/*
	pushurl = agency-admin:scratch/agency/repos/git-deploy-test-staging.agency.com.git
	pushurl = git@gitlab.com:agency/git-deploy-test.git
[branch "staging"]
	remote = staging
	merge = refs/heads/staging
```

Each repo on the admin server is configured with a git hook which will execute after receiving files pushed to it.
Example of a `post-receive` git hook:
```
#!/bin/sh
#
# post-receive to deploy code after receiving a push
# goes in .git/hooks/ on the admin server

# Source (bashism) HOSTNAME REPOURL BRANCH WEBROOT OWNER
. "./hooks/git-hooks-env"
if [ ! $? -eq 0 ]; then
	echo "Environment file not found, exiting."
	exit 1
fi
WORKING_DIRECTORY="/var/wordpress-deploy"
REPOS="$WORKING_DIRECTORY/repos"
WORKTREES="$WORKING_DIRECTORY/worktrees"
REPO="$REPOS/$HOSTNAME.git"
WORKTREE="$WORKTREES/$HOSTNAME"
git --work-tree=$WORKTREE --git-dir=$REPO checkout -f $BRANCH
if [ ! $? -eq 0 ]; then
	notify "git hook failed to checkout -f for $HOSTNAME" critical
	exit 1
fi

export WORKTREES
wordpress-deploy $HOSTNAME $WEBROOT $OWNER

exit 0
```

In the `post-receive` example above you can see that repo-specific variables are sourced from a file called `git-hooks-env`.
The repos are separated from the worktrees, to keep all the git stuff segregated from the actual codebase.
Both the repository and worktree paths are all named after the full.ain of the deployment target server ($HOSTNAME) for predictability.
The (https://gitlab.com/neilscudder/script-generator)[script-generator] generates an initialization script based on a CSV.

Example admin server initialization script:
```
#!/bin/bash

# Initialize all repos not already initialized

WORKING_DIRECTORY="/var/wordpress-deploy"
REPOS="$WORKING_DIRECTORY/repos"
WORKTREES="$WORKING_DIRECTORY/worktrees"

if [ ! -d $REPOS ]; then
	mkdir $REPOS || exit 1
fi
if [ ! -d $WORKTREES ]; then
	mkdir $WORKTREES || exit 1
fi

git-deploy-init(){
	DOMAIN="$1"
	REPOURL="$2"
	BRANCH="$3"
	WEBROOT="$4"
	OWNER="$5"
	REPO="$REPOS/$DOMAIN.git"
	WORKTREE="$WORKTREES/$DOMAIN"
	if [ -d $REPO ]; then
		echo "$REPO already exists, skipping."
	else
		git clone --bare "$REPOURL" "$REPO"
		chgrp developers -R "$REPO"
		chmod g+s -R "$REPO"
	fi
	if [ -d $WORKTREE ]; then
		echo "$WORKTREE already exists, skipping."
	else
		mkdir "$WORKTREE"
		chgrp developers -R "$WORKTREE"
		chmod g+s -R "$WORKTREE"
	fi

	cp post-receive $REPO/hooks/
	chmod a+x $REPO/hooks/post-receive

	echo "# Git Hooks Environment
HOSTNAME=$DOMAIN
BRANCH=$BRANCH
WEBROOT=$WEBROOT
WEBROOT_OWNER=$OWNER" > $REPO/hooks/git-hooks-env
}

git-deploy-init git-deploy-test.agency.com git@gitlab.com:agency/git-deploy-test.git main /var/www/html www-data
git-deploy-init git-deploy-test-staging.agency.com git@gitlab.com:agency/git-deploy-test.git staging /var/www/html www-data

exit 0
```

