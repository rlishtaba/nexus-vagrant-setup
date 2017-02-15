# Nexus server demo

Simple linux based server capable of running JVM based Nexus server.

## Usage

##### Requirements

- Vagrant
- Virtual Box

##### Test your local dependencies

    % VBoxManage --version

    % vagrant --version

If don't have declared dependencies - go get them.    

### Bootstrap

    # clone sources via HTTPS with username and password   
    % git clone http://<username>@almgit.ncr.com/scm/~rl185160/nexus-demo.git

    # clone sources via SSH
    % git clone ssh://git@almgit.ncr.com:7999/~rl185160/nexus-demo.git

    % cd nexus-demo

    % vagrant up

### Usage

Once vagrant finished with bootstrap, let Nexus warm up, this is JVM based app
and it is require ~2 minutes of your patience to preload JVM libs.

 Open your browser and visit ```http://nexus.ncr.systems```

 ### Credentials

 Credentials are defaulting to the following

    username: admin
    password: admin123

### Contributing

Do whatever you want, but please, do not try to push back altered source.
