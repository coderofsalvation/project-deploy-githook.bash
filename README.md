project-deploy-githook.bash
========================

<img alt="" src="pdg.png"/>

Want hasslefree deployment of multiple web applications in one VPS or docker container?

pdg means KISS automatic web deployment for VPS (minimalist PAAS), web project bootstrapper in few lines of bash.

> Zero dependencies: only bash git and ssh!

## Usage

> NOTE: for nodejs applications use [this variant](https://github.com/coderofsalvation/nodejs-deploy-githook.bash)

Download & configure pdg on liveserver:

    $ ssh foo@liveserver.com 
    $ pdg config repositories_dir /srv/webrepos    # location of gitrepos
    $ pdg config apps_dir /srv/webapps             # where apps run

Yay! now we can remotely bootstrap web-projects:

    $ ssh foo@liveserver.com pdg init fooproject 8111

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

    $ git clone foo@liveserver.com:/srv/webrepos/fooproject

    ...(code and commit)..

    $ git push origin master

    ...(yay! app is deployed, npm packages installed/updated, infinite application-loop started..)

Your repo will contain a '.pdg'-folder with extra deploymenthooks..for free!

## Manage remotely

    $ ssh foo@liveserver.com pdg status fooproject
    app fooproject is running
    $ ssh foo@liveserver.com pdg app list
    pdg> fooproject
    $ ssh foo@liveserver.com pdg app stop fooproject
    $ ssh foo@liveserver.com pdg app start fooproject
    $ ssh foo@liveserver.com pdg app restart fooproject
    $ ssh foo@liveserver.com pdg app delete fooproject

## Remote logging:

    $ ssh foo@liveserver.com tailf /srv/webrepos/fooproject/nohup.out
    trigger .pdg/hooks/stop
    Tue Apr 28 08:53:55 CEST 2015 stopping /srv/webapps/fooproject (pid 27474)
    trigger .pdg/hooks/build
    trigger .pdg/hooks/patch
    trigger .pdg/hooks/test
    Tue Apr 28 08:53:58 CEST 2015 starting /srv/webapps/fooproject at port 8111

## App start during server reboot

Just put this somewhere in an /etc/init.d/ script:

    pdg app status projectfoo || pdg app start projectfoo

All apps at once:

    for app in /srv/webapps/*; do 
      appname=$(basename $app)
      pdg app status $appname || pdg app start $appname
    done
