Deployment is handled by the `wordpress-deploy` script which is called by a git hook which resides in a git remote on an internal administration server.
In a typical situation code is pushed to a single `origin` remote.
To use `wordpress-deploy` we are going to associate two separate `push` URLs per remote, and set specific remotes for branches which have servers associated with them.

Assumptions:
- the `origin` remote is at gitlab
- you have already cloned the repo and changed to the local repo directory
- the `main` branch gets deployed to the production server
- the `staging` branch deploys to a staging server
- the `development` branch.s not deploy anywhere
- each remote on the admin server is associated with an individual branch and a single deployment target (this configuration is automated with the `script-generator`)

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


