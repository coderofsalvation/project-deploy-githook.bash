project-deploy-githook.bash
========================

<img alt="" src="pdg.png"/>

Want hasslefree deployment of multiple web applications in one VPS or docker container?

pdg means KISS automatic web deployment for VPS (minimalist PAAS), web project bootstrapper in few lines of bash.

> Zero dependencies: only bash git and ssh!

## Usage

> NOTE: for nodejs applications use [this variant](https://github.com/coderofsalvation/nodejs-deploy-githook.bash)

Download pdg on liveserver:

    liveserver$ ssh foo@liveserver.com 
    liveserver$ sudo git clone https://github.com/coderofsalvation/project-deploy-githook.bash.git /opt/pdg 
    liveserver$ sudo ln -s /opt/pdg/pdg /usr/local/bin/pdg

Configure pdg:

    liveserver$ pdg config repositories_dir /srv/webrepos    # location of gitrepos
    liveserver$ pdg config apps_dir /var/www                 # where webapps run

Yay! now we can remotely bootstrap web-projects on the liveserver:

    local$ ssh foo@liveserver.com pdg init fooproject

    pdg> Initialized empty Git repository in /srv/webrepos/fooproject
    pdg> --- initing repo
    pdg> --- initing .pdg/hooks/*
    pdg> --- committing README.md, LICENSE and pdg hooks
    pdg> [master (root-commit) 1870848] 1st commit
    pdg> 5 files changed, 10 insertions(+)
    pdg> create mode 100644 .gitignore
    pdg> create mode 100755 .pdg/hooks/build
    pdg> create mode 100755 .pdg/hooks/patch
    pdg> create mode 100755 .pdg/hooks/test
    pdg> create mode 100755 .pdg/hooks/start
    pdg> create mode 100755 .pdg/hooks/stop
    pdg> create mode 100644 LICENSE
    pdg> create mode 100644 README.md
    pdg> create mode 100644 package.json
    pdg> --- pushing to origin
    pdg> get    repo:    git clone foo@liveserver.com:/srv/webrepos/fooproject    
    pdg> logview app:    ssh foo@liveserver.com tailf /srv/webrepos/fooproject/nohup.out
    pdg> deploy  app:    git push origin master

## Code locally, deploy to remote

    local$ git clone foo@liveserver.com:/srv/webrepos/fooproject

start coding: 

    local$ cd fooproject 
    local$ echo "hello world" > index.html
    local$ php composer.phar init
    local$ php composer.phar require monolog/monolog
    local$ git add index.html composer.json

deploy!: 

    local$ git commit -m "1st commit"
    local$ git push origin master

yay! app is deployed, composer/npm packages installed/updated:

     1 file changed, 5 insertions(+)
     create mode 100644 composer.json
    remote: trigger .pdg/hooks/stop
    remote: HEAD is now at 96c5e9d 1st commit
    remote: From /tmp/repos/testapp
    remote:  * branch            master     -> FETCH_HEAD
    remote:    96c5e9d..738cb3d  master     -> origin/master
    remote: Updating 96c5e9d..738cb3d
    remote: Fast-forward
    remote:  index.html    | 5 +++++
    remote:  composer.json | 5 +++++
    remote:  1 file changed, 5 insertions(+)
    remote:  create mode 100644 composer.json
    remote: trigger .pdg/hooks/build
    remote: #!/usr/bin/env php
    remote: All settings correct for using Composer
    remote: Downloading...
    remote: 
    remote: Composer successfully installed to: /tmp/apps/testapp/.pdg/composer.phar
    remote: Use it: php .pdg/composer.phar
    remote: Loading composer repositories with package information
    remote: Updating dependencies (including require-dev)
    remote:   - Installing monolog/monolog (1.0.2)
    remote:     Loading from cache
    remote: 
    remote: Writing lock file
    remote: Generating autoload files
    remote: trigger .pdg/hooks/patch
    remote: trigger .pdg/hooks/test
    To /tmp/repos/testapp
       96c5e9d..738cb3d  master -> master


Your repo will contain a '.pdg'-folder with extra deploymenthooks..for free!

## Manage remotely

    local$ ssh foo@liveserver.com pdg status fooproject
    app fooproject is running
    local$ ssh foo@liveserver.com pdg app list
    pdg> fooproject
    local$ ssh foo@liveserver.com pdg app stop fooproject
    local$ ssh foo@liveserver.com pdg app start fooproject
    local$ ssh foo@liveserver.com pdg app restart fooproject
    local$ ssh foo@liveserver.com pdg app delete fooproject

## Remote logging:

    local$ ssh foo@liveserver.com tailf /srv/webrepos/fooproject/nohup.out
    trigger .pdg/hooks/stop
    Tue Apr 28 08:53:55 CEST 2015 stopping /srv/webapps/fooproject (pid 27474)
    trigger .pdg/hooks/build
    trigger .pdg/hooks/patch
    trigger .pdg/hooks/test
    Tue Apr 28 08:53:58 CEST 2015 starting /srv/webapps/fooproject at port 8111

## Optional: App start during server reboot

*.pdg/hooks/start* and *.pdg/hooks/stop* can be used to start/stop your application.

So you could just put this somewhere in an /etc/init.d/ script:

    pdg app status projectfoo || pdg app start projectfoo

All apps at once:

    for app in /srv/webapps/*; do 
      appname=$(basename $app)
      pdg app status $appname || pdg app start $appname
    done
