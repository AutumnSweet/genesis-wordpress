### !!! End Of Life Notice !!!
> Aside from occasional bug fixes or security patches, Genesis is no longer actively supported.
> 
> Its successor, [Evolution](https://github.com/evolution/wordpress), has a similar feature set, as well as [a built-in importer](https://github.com/evolution/wordpress/blob/master/docs/TUTORIAL-IMPORT.md) for existing Wordpress projects.

### !!! Repository Move Notice !!!
> This project was recently moved across organizations in github. In order to continue using it, you'll have to update the generator:
>
> `npm update generator-genesis-wordpress -g`

# Genesis WordPress

> Rapidly create, develop, & deploy WordPress across multiple environments.
> ![Genesis WordPress Demo](demo.gif)

## Features

- Generate a functional WordPress site + server
- First-class local development
- Independently stage features for review
- Use production data when developing
- High-performance, zero-configuration caching out of the box
- Easily monitor remote server errors
- Instant, secure SSH access
- Automated server provisioning
- Consistent, reliable environments


## Installation

Ensure you have the latest versions of [NodeJS][9] **v0.10**, [Vagrant v1.6.*](http://www.vagrantup.com/downloads.html), & [VirtualBox v.4.2.*](https://www.virtualbox.org/wiki/Download_Old_Builds_4_2).

### Scaffolding & Development

Install [Yeoman][2] **v1**, [Bower][6] **v1.3.3+**, [Genesis WordPress Generator][1], & [Vagrant Host Manager][4]:

    $ npm install -g yo bower generator-genesis-wordpress
    $ vagrant plugin install vagrant-hostmanager

If you get EMFILE issues, try running: `$ ulimit -n 4096`.

*(You can check your versions by running `node --version`, `npm --version`, etc.)*

### Deployment

Install [Capistrano v2.15.*][5] via [Bundler][1] & [Ansible][7]:

    $ sudo bundle install
    $ sudo easy_install pip
    $ sudo pip install ansible


## Getting Started


## Step 1 – Creating or Upgrading a Site

*Use the [Genesis WordPress Generator][1] for scaffolding.*


## Step 2 – Working Locally

First, ensure you're using the latest version of [Genesis WordPress][0] with [Bower][6]:

    $ bower update

Next, use [Vagrant][3] to create & provision your local environment:

    $ vagrant up

Now open http://local.mysite.com (or whatever your site's domain name is)!

If the site doesn't load for you, you may have to manually
provision your local machine:

    $ vagrant provision

Or, update your local `/etc/hosts` with [Vagrant Host Manager][4]:

    $ vagrant hostmanager

Finally, if things worked while you were at the office but broke when you got home, you probably need to just get Vagrant a new IP address:

    $ vagrant reload


## Step 3 – Wrapping Up

When you're done working on your site, suspend the VM to save on CPU & memory:

    $ vagrant suspend

You can destroy the VM entirely (while keeping your local files) to save on disk space:

    $ vagrant destroy


## Deployment

First, ensure your project on Github can be accessed by remote servers.  To do this,
access the project's *Settings -> Deploy Keys* in Github and add `provisioning/files/ssh/id_rsa.pub`.

Next, assuming the server has been provisioned, deploy your code on Github:

    $ bundle exec cap production deploy

The latest code is now live:

    > http://production.mysite.com/

If you deploy to `staging`, the name of the current branch (e.g. `my-feature`) is deployed:

    > http://my-feature.staging.mysite.com/

In the rare event the changes weren't supposed to go live, you can rollback to the previous release:

    $ bundle exec cap production deploy:rollback

**Note that deployments use the project's *Github repository* as the source, not your local machine!**


## Syncing Files/Database

### From Local to Remote

Suppose you have just provisioned & deployed to a new server, but the site obviously won't work without
a database or uploaded images.

You can **overwrite the remote database** with your local VM's:

    $ bundle exec cap production genesis:up:db

You can sync your local files to the remote filesystem:

    $ bundle exec cap production genesis:up:files

Or, you can perform both actions together:

    $ bundle exec cap production genesis:up

Once a site is live, you *rarely* need to sync anything up to the remote server.  If anything,
you usually sync changes *down*.

In the rare case you need to _destructively_ sync files to the remote server (meaning, delete files on remote that don't exist locally), there is a special use command:

    $ bundle exec cap production genesis:up:mirror

### From Remote to Local

Suppose you have a live site that you need to work on locally.  Like the previous section,
you can sync down the database, the files (e.g. uploaded images), or both:

    $ bundle exec cap production genesis:down:db
    $ bundle exec cap production genesis:down:files
    $ bundle exec cap production genesis:down


## Provisioning

The following environments are expected to exist and resolve via DNS to simplify deployment & provisioning:

- `local` (e.g. http://local.mysite.com)
- `staging` (e.g. http://staging.mysite.com/, http://my-feature.staging.mysite.com/)
- `production` (e.g. http://production.mysite.com/, http://www.mysite.com/, http://mysite.com/)

If you're deploying to a new machine (e.g. production.mysite.com), you first need to provision it:

    $ bundle exec cap production genesis:provision

If there is an error, you may be prompted to re-run the command with an explicit username/password:

    $ bundle exec cap production genesis:provision -S user=myuser -S password=mypassword

*From that point on, tasks will use a private key (`provisioning/files/ssh/id_rsa`).*

In the event you already have a live site, you can modify the settings in `deployment/stages/old.rb` to
migrate the old server to a new server:

    # Start the local VM
    $ vagrant up

    # Provision the new server
    $ bundle exec cap production provision
    $ bundle exec cap production deploy

    # Download the old site to local
    $ bundle exec cap old genesis:down

    # Upload the old site to production
    $ bundle exec cap production genesis:up

Now you can switch DNS for http://www.mysite.com/ to point to http://production.mysite.com/'s IP!

## Genesis Tasks

Most of the functionality regarding remote servers are handled by custom [Capistrano][5] tasks,
which you can see by running:

    $ bundle exec cap -T genesis
    cap genesis:down        # Downloads both remote database & syncs remote files into Vagrant
    cap genesis:down:db     # Downloads remote database into Vagrant
    cap genesis:down:files  # Downloads remote files to Vagrant
    cap genesis:logs        # Tail Apache error logs
    cap genesis:permissions # Fix permissions
    cap genesis:provision   # Runs project provisioning script on server
    cap genesis:restart     # Restart Apache + Varnish
    cap genesis:ssh         # SSH into machine
    cap genesis:start       # Start Apache + Varnish
    cap genesis:stop        # Stop Apache + Varnish
    cap genesis:up          # Uploads Vagrant database & local files into production
    cap genesis:up:db       # Uploads Vagrant database into remote
    cap genesis:up:files    # Uploads local project files to remote
    cap genesis:up:mirror   # **DESTRUCTIVELY** overrides files on remote
    cap genesis:teardown    # Remove any existing remote deployment files; counterpart to cap's built-in deploy:setup

Now run any one of those commands against an environemnt:

    $ bundle exec cap local genesis:restart

## Troubleshooting

### SSH - Prompting for a password

If you're seeing this:

    $ bundle exec cap staging genesis:ssh
    deploy@staging.example.com's password:

Then the `deploy` user's ssh keys on your remote server *do not match* the keys in your local repository.

You should first ensure that your local repository is up to date, thereby ensuring you are using the latest versioned ssh keys.

    $ git checkout master
    $ git pull origin master
    $ bundle exec cap staging genesis:ssh

If the problem persists, this means that the keys on your remote server are out of date or otherwise incorrect, and you must re-provision by specifying a username and password:

    $ bundle exec cap staging genesis:provision -S user=userWithRootOrSudoAccess -S password=usersHopefullyStrongPassword

### SSH - Host key mismatch

If you're seeing this:

    $ bundle exec cap staging genesis:ssh
    @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
    @    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
    @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
    IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
    Someone could be eavesdropping on you right now (man-in-the-middle attack)!
    It is also possible that a host key has just been changed.
    The fingerprint for the RSA key sent by the remote host is
    d3:4d:b4:4f:d3:4d:b4:4f:d3:4d:b4:4f:d3:4d:b4:4f.
    Please contact your system administrator.
    Add correct host key in ~/.ssh/known_hosts to get rid of this message.
    Offending RSA key in ~/.ssh/known_hosts:68
    RSA host key for staging.example.com has changed and you have requested strict checking.
    Host key verification failed.

Then you have at least one existing entry in your `~/.ssh/known_hosts` file (indicated, in the example above, to be on line 68), with a *different* key than the server is returning.

You can search for all line(s) matching the server name and/or ip address using `grep`:

    $ cat ~/.ssh/known_hosts | grep -n "staging.example.com"
    68:staging.example.com,192.168.1.42 ssh-rsa AAAAB3NzaCd34db33f...

Now, remove those lines from said file, using your text editor of choice.

### SSH - Permission denied (publickey)

If you're seeing this:

```
    servers: ["production.yourwebsite.com"]
    [production.yourwebsite.com] executing command
 ** [production.yourwebsite.com :: out] Permission denied (publickey).
 ** [production.yourwebsite.com :: out] fatal: The remote end hung up unexpectedly
```

Then you probably need to add the SSH keys to your GitHub repo. Open `provisioning/files/ssh/id_rsa.pub` and copy/paste the entire contents (the ssh-rsa key) to your repo by visiting __Settings > Deploy Keys > Add deploy key__.

For more help on this, refer to the [GitHub Docs](https://help.github.com/articles/error-permission-denied-publickey).

### SSH - SSH Authentication Failed!

If you're seeing this:

```
SSH authentication failed! This is typically caused by the public/private
keypair for the SSH user not being properly set on the guest VM. Please
verify that the guest VM is setup with the proper public key, and that
the private key path for Vagrant is setup properly as well.
```

Then you're probably missing the [Vagrant Public](https://raw.githubusercontent.com/mitchellh/vagrant/master/keys/vagrant.pub) Key in your `authorized_keys`. To add it run:
`curl https://raw.githubusercontent.com/mitchellh/vagrant/master/keys/vagrant.pub >> ~/.ssh/authorized_keys`

### Vagrant - Error While Executing `VBoxManage`

If you're seeing this:

```
There was an error while executing `VBoxManage`, a CLI used by Vagrant
for controlling VirtualBox. The command and stderr is shown below.

Command: ["hostonlyif", "create"]
```

The you'll need to restart VirtualBox with:
```
sudo /Library/StartupItems/VirtualBox/VirtualBox restart
```

### Vagrant - Error changing hostname

If you're seeing this:
```
==> local: Setting hostname...
Vagrant attempted to execute the capability 'change_host_name'
on the detect guest OS 'linux', but the guest doesn't
support that capability. This capability is required for your
configuration of Vagrant. Please either reconfigure Vagrant to
avoid this capability or fix the issue by creating the capability.
```

You've hit a [known issue](https://github.com/mitchellh/vagrant/issues/7625) with vagrant 1.8.5 (the latest version at the time of this writing). You will need to [patch vagrant as described here](http://stackoverflow.com/questions/38636023/vagrant-not-supported-the-capability-change-host-name#answer-38642003), or wait for the upcoming 1.8.6 release.

### Bower - ENOENT or ENOEMPTY error during deploy

If you're seeing this:

```
 ** [out :: production.yourwebsite.com] bower ENOTEMPTY     ENOTEMPTY, rename '/tmp/deploy/bower/jquery-14131-V6RuH0'
 ** [out :: production.yourwebsite.com] 
 ** [out :: production.yourwebsite.com] Stack trace:
 ** [out :: production.yourwebsite.com] Error: ENOTEMPTY, rename '/tmp/deploy/bower/jquery-14131-V6RuH0'
 ** [out :: production.yourwebsite.com] 
```

You've been bitten by [a known concurrency bug](https://github.com/bower/bower/issues/933), and need to update bower on your remote server to **1.3.3** or higher:

```
npm install -g bower
```

### Apt - Error While Executing `apt-get -f install`

If you're seeing this:

```
 ** [out :: staging.yourwebsite.com] E: Unmet dependencies. Try 'apt-get -f install' with no packages (or specify a solution).
 ** [out :: staging.yourwebsite.com] ERROR: hostname is not a legal parameter in an Ansible task or handler
```

You likely have a full `/boot` partition, which is preventing Ubuntu's package manager from functioning normally. To fix this, you'll need to remove outdated linux kernels -- **this must be done _on the remote server_**.

You may [go through the steps manually](http://askubuntu.com/questions/345588/what-is-the-safest-way-to-clean-up-boot-partition#answer-430944), or use a python script that must be run with root or sudo permissions.

```
wget -O /tmp/purge-kernels.py https://raw.githubusercontent.com/EvanK/ubuntu-purge-kernels/master/purge-kernels.py
sudo python /tmp/purge-kernels.py
```

Afterward, you should have `apt` and `dpkg` clean up after themselves, update the grub kernel list, and reboot the server:

```
sudo apt-get -f install
sudo apt-get -f autoremove
sudo update-grub
sudo reboot now
```

### MySQL - Abnormal disk space usage by InnoDB

Due to a [known issue for MySQL 5.5 or below](https://github.com/evolution/genesis-wordpress/issues/160), the innodb storage engine will consume but never release disk space by default:

```
deploy@production:~$ sudo ls -alh /var/lib/mysql/ib{data1,_logfile0,_logfile1}
-rw-rw----  1 mysql mysql 1.6G Aug 29 12:50 ibdata1
-rw-rw----  1 mysql mysql 5.0M Aug 29 12:50 ib_logfile0
-rw-rw----  1 mysql mysql 5.0M Aug 29 12:44 ib_logfile1
```

This issue is resolved with a change to `my.cnf` (in the provisioning provided by genesis v0.3.2), and the following steps:


1. sync down a copy of your target database (	`production` in this example:
  * `bundle exec cap production genesis:down:db`

2. reprovision target stage, to update `my.cnf` (with genesis v0.3.2 or higher):
  * `bundle exec cap production genesis:provision`

3. ssh in: `bundle exec cap production genesis:ssh`
  * stop mysql service: `sudo service mysql stop`
  * remove innodb files: `sudo rm /var/lib/mysql/ib{data1,_logfile0,_logfile1}`
  * restart mysql service: `sudo service mysql start`

4. reprovision a second time, to recreate wordpress db and user:
  * `bundle exec cap production genesis:provision`

5. sync up your db copy:
  * `bundle exec cap production genesis:up:db`

You _may_ also need to restart services afterward: `bundle exec cap production genesis:restart`

## Changelog

- v0.3.2 - Fixed mysql innodb disk space issues ([#160](https://github.com/evolution/genesis-wordpress/pull/160))
- v0.3.1 - Fixed php warning from HTTPS check in Genesis.php ([#159](https://github.com/evolution/genesis-wordpress/pull/159))
- v0.3.0 - Added distro & version check within provisioning ([#158](https://github.com/evolution/genesis-wordpress/pull/158))
- v0.2.68 - Fixed bug regarding undefined `aliases` variable
- v0.2.67 - Added rudimentary ssl reverse proxy support ([#157](https://github.com/evolution/genesis-wordpress/pull/157))
- v0.2.66 - Added domain aliases ([#156](https://github.com/evolution/genesis-wordpress/pull/156))
- v0.2.65 - Moved across organizations in github and updated urls
- v0.2.64 - Added ESI cache ttl support ([#155](https://github.com/evolution/genesis-wordpress/pull/155))
- v0.2.63 - Added bypass for git port ([#154](https://github.com/evolution/genesis-wordpress/pull/154))
- v0.2.62 - Improved apache log rotation ([#152](https://github.com/evolution/genesis-wordpress/pull/152))
- v0.2.61 - Added db backup before syncing up to remote stages ([#151](https://github.com/evolution/genesis-wordpress/pull/151))
- v0.2.60 - Fixed rsync copy-links/keep-dirlinks bug in `up:mirror` command
- v0.2.59 - Added `genesis:up:mirror` command ([#149](https://github.com/evolution/genesis-wordpress/pull/149))
- v0.2.58 - Fixed Vagrant 1.7.0+ ssh key issue, and fixed unit tests
- v0.2.57 - Fixed generator htaccess issue
- v0.2.56 - Hardened WP Security ([#143](https://github.com/evolution/genesis-wordpress/issues/143)), fixed tasksel issue ([#144](https://github.com/evolution/genesis-wordpress/pull/144))
- v0.2.55 - `genesis:provision` works despite `Net:SSH` failures ([#131](https://github.com/evolution/genesis-wordpress/pull/131))
- v0.2.54 - Rewrite supports HTTPS ([#101](https://github.com/evolution/genesis-wordpress/issues/101))
- v0.2.53 - Introduce `genesis:up:limited` for rsync'ing only the shared folders ([#80](https://github.com/evolution/genesis-wordpress/pull/80))
- v0.2.52 - Sensible Apache2 defaults ([#116](https://github.com/evolution/genesis-wordpress/pull/116))
- v0.2.51 - Add `Gemfile` to generator ([#126](https://github.com/evolution/genesis-wordpress/pull/126))
- v0.2.50 - Fix duplication of `genesis:backup` and `genesis:down` ([122](https://github.com/evolution/genesis-wordpress/pull/122))
- v0.2.49 - Fix variable sustitution in `provision.yml` [f334764](https://github.com/evolution/genesis-wordpress/commit/f334764ad5e36ef847fe6752fb43cc553b74fde4)
- v0.2.48 - Fix for `$` in passwords ([113](https://github.com/evolution/genesis-wordpress/pull/113))
- v0.2.47 - Automatically `cleanup` after `deploy` ([112](https://github.com/evolution/genesis-wordpress/pull/112))
- v0.2.46 - Legacy variable substitution([109](https://github.com/evolution/genesis-wordpress/pull/109))
- v0.2.45 - Add `genesis:backup:db` command ([120](https://github.com/evolution/genesis-wordpress/pull/120))
- v0.2.44 - Explicitly bypassing bower interactive prompts during deployment
- v0.2.43 – Revert `5afaf80`, fixes v0.2.38
- v0.2.42
    - `auto_correct` Vagrant SSH port
    - Default to latest WordPress
    - Add tests ([#69](https://github.com/evolution/genesis-wordpress/pull/96))
- v0.2.41 – Fix Varnish cookie bug ([#90](https://github.com/evolution/genesis-wordpress/pull/90))
- v0.2.40 – Set hostname on each machine ([#45](https://github.com/evolution/genesis-wordpress/pull/45))
- v0.2.39 – Revert v0.2.37 (aa9e83f)
- v0.2.38 – Move events to after `deploy:update_code` ([#82](https://github.com/evolution/genesis-wordpress/pull/82))
- v0.2.37 – Fix issues with Varnish ([#62](https://github.com/evolution/genesis-wordpress/pull/62):
    - Cleaned up cookie logic in `production.vcl` (see #28, and 3fd9d0c)
    - Fixed wp cookie check in `receive/wordpress.vcl` (see 9c2f358)
    - Changed varnish to file backend (see #53)
    - Removed cache bypassing for local env (see fa96873)
    - Removed caching of static files (see 99eb9ad)
    - Piping `wp-(login|admin)` instead of passing (see 89cb137)
- v0.2.36 – Add `postfix` ([#72](https://github.com/evolution/genesis-wordpress/pull/72))
- v0.2.35 – Add `genesis:teardown` ([#55](https://github.com/evolution/genesis-wordpress/pull/55)) & fix `date.timezone` ([#73](https://github.com/evolution/genesis-wordpress/pull/73))
- v0.2.34 – Default to WordPress 3.7.1 ([#74](https://github.com/evolution/genesis-wordpress/pull/74))
- v0.2.33 – Allow two-part TLDs ([#77](https://github.com/evolution/genesis-wordpress/issues/77https://github.com/evolution/genesis-wordpress/issues/77))
- v0.2.32 – Fix issue with adding `deploy` user to `www-data` group ([#70](https://github.com/evolution/genesis-wordpress/pull/70))
- v0.2.31 – Attempt to fix issues with `genesis:permissions` ([#54](https://github.com/evolution/genesis-wordpress/pull/54))
- v0.2.30
    - Run `vagrant up` prior to `genesis:up:db` and `genesis:down:db` ([#59](https://github.com/evolution/genesis-wordpress/pull/59))
    - Use VirtualBox's `natdnshostresolver1` to resolve DNS ([#65](https://github.com/evolution/genesis-wordpress/pull/65/files))
    - [Ensure SSH port is not an octet](https://github.com/evolution/genesis-wordpress/pull/66)
- v0.2.29 – [Apache + PHP performance tuning](https://github.com/evolution/genesis-wordpress/pull/64)
- v0.2.28 – Update with last PRs from Genesis WordPress Generator
- v0.2.27 – Awwww snap!! Making it so the [Genesis WordPress Generator](https://github.com/genesis/generator-wordpress) is always up-to-date!
- v0.2.26 – Use `sudo` instead of `invoke_command` ([#41](https://github.com/evolution/genesis-wordpress/issues/41))
- v0.2.25 – Directories are now `775` and owned by `deploy:www-data` ([#31](https://github.com/evolution/genesis-wordpress/issues/31))
- v0.2.24 – Set Varnish & PHP to `512M`
- v0.2.23 – Only bypass for logged in users, not logged out
- v0.2.22 – Bypass cache for logged in users ([#19](https://github.com/evolution/genesis-wordpress/pull/19))
- v0.2.21 – Run genesis:permissions on server, not local!
- v0.2.20 – Fix `genesis:permissions`
- v0.2.19 – Fix permissions after `deploy` & `genesis:files:up`
- v0.2.18 – Remove pretty_print.  **VERBOSE ERRORS FTW!!!**
- v0.2.17 – Add `curl` as default module
- v0.2.17 – Don't sync `.sql` files by default
- v0.2.16 – `chmod 600` the ssh key only if it exists
- v0.2.15 – Rename production logs from `www-` to `production-`
- v0.2.14 – Localize `wp_get_attachment_url`
- v0.2.13 – `chmod 600` the ssh key when running `genesis:down/up`
- v0.2.12 – Remove `genesis:restart` after `genesis:down:*`
- v0.2.11 – Fix URLs for uploads & permalinks
- v0.2.10 – Fix get_option( 'home' )
- v0.2.9 – Remove probe for `/server-status`
- v0.2.8 – Fix local access log
- v0.2.7 – Restart after all `genesis:down:*` and `genesis:up:*`
- v0.2.6 – Add priority to vhosts
- v0.2.5 – Set deploy shell to `/bin/bash`
- v0.2.4 – `genesis:restart` runs on all `genesis:up` commands
- v0.2.3 – Fix bug with static assets being cached by Varnish
- v0.2.2 – Fix bug when inferring `:branch`
- v0.2.1 – Fix bug when `git branch` returns nothing
- v0.2.1 – Remove Varnish error pages
- v0.2.0 – Rename `genesis:tail` to `genesis:logs`
- v0.1.21 – Bypass Varnish for `4xx` & `5xx` error codes
- v0.1.20 – Bypass Varnish for `local.`, `wp-login`, and `wp-admin`
- v0.1.20 – Run `genesis:restart` after `deploy:restart`
- v0.1.19 – Add Varnish to `restart`, `start`, `stop`
- v0.1.18 – Initial Varnish
- v0.1.17 – Add `shared_children`
- v0.1.16 – `chmod 600 id_rsa`
- v0.1.15 – Sync with generator-genesis-wordpress#`0.1.6`
- v0.1.14 – Re-order NodeJS installation
- v0.1.13 – Update cache before NodeJS
- v0.1.12 – Bad ansible command (NodeJS)
- v0.1.11 – Forgot to install NodeJS
- v0.1.10 – Attempt to install NodeJS + Bower
- v0.1.9 – Fix `v0.1.8`
- v0.1.8 – Add filter for `option_siteurl` to fix redirects in `wp-admin`
- v0.1.7 – Fix `ssh` & remove `WP_SITEURL`
- v0.1.6 – Rename `wp` capistrano task namespace to `genesis`
- v0.1.5 – Add `cache` to `rsync_exclude` folders
- v0.1.4 – Bower release

## License

[MIT License](http://en.wikipedia.org/wiki/MIT_License)

[0]: https://github.com/evolution/genesis-wordpress/
[1]: https://github.com/genesis/generator-wordpress/
[2]: http://yeoman.io/
[3]: http://www.vagrantup.com/
[4]: https://github.com/smdahlen/vagrant-hostmanager
[5]: https://github.com/capistrano/capistrano/wiki/2.x-Getting-Started
[6]: http://bower.io/
[7]: http://www.ansibleworks.com/
[8]: https://www.virtualbox.org/
[9]: http://nodejs.org/
[10]: http://bundler.io/

[![Bitdeli Badge](https://d2weczhvl823v0.cloudfront.net/genesis/wordpress/trend.png)](https://bitdeli.com/free "Bitdeli Badge")

