### Background
These instructions are to install Galaxy and the HCSvLab tools on a CentOS machine. This can easily be adapted to other distributions.

The Galaxy instructions are based on the following from the Galaxy website
* [Basic install](http://wiki.galaxyproject.org/Admin/Get%20Galaxy) 
* [Running in production](http://wiki.galaxyproject.org/Admin/Config/Performance/ProductionServer)

### Assumptions
* You have a fresh CentOS machine. 
* You have a non-root user account on this machine with sudo privileges. We used "galaxy".
* You are logged in as this user and are in the home directory.

### Setup
Install development tools

    sudo yum groupinstall "Development Tools"

Install mercurial

    sudo yum install mercurial

Pull down Galaxy from mercurial and select the stable branch

    hg clone https://bitbucket.org/galaxy/galaxy-dist/
    cd galaxy-dist
    hg update stable

[Install NLTK](NLTK.md) (you should install this as the same user that will run Galaxy)

[Install the Johnson Charniak Parser](JCParser.md) (you should install this as the same user that will run Galaxy)

[Install the JC Parser NLTK wrapper](JCPNLTKWrapper.md) (you should install this as the same user that will run Galaxy)

Install the following tools (required for displaying parse trees)
    
    sudo yum install tkinter python-matplotlib-tk.x86_64 xorg-x11-server-Xvfb ImageMagick
    Xvfb :1 -screen 0 1024x768x24 &
    export DISPLAY=:1
    sudo yum install xhost
    xhost +
    
Add the `DISPLAY` configuration to your bashrc

    cd ~
    vi .bashrc
        // Paste the following line into the file:
        export DISPLAY=:1
    source ~/.bashrc

Install HCSvLab Galaxy customisations and tool wrappers

    cd ~/galaxy-dist
    git init
    git remote add origin git://github.com/IntersectAustralia/hcsvlab-galaxy.git
    git pull origin master
    chmod 755 galaxy
    
Modify the start script with user and path - modify the `RUN_AS` and `RUN_IN` to suit your path and user    

    vi galaxy
    
Start Galaxy    

    ./galaxy start
    
### Production Configuration    
**If you are just trying it out or running it locally for development, you can stop here.** If running in production, we recommend the following steps (which come from the Galaxy recommendations for running in production). We suggest you review the Galaxy documentation and choose options that are most appropriate to your local installation. Below is what we did.

Turn off development settings

Edit `~/galaxy-dist/universe_wsgi.ini` and modify as follows:

    debug = False
    use_interactive = False

Install Postgres, create a Postgres user called "galaxy" and create a database called "galaxy". If you need guidance on this see [Install Postgres](Postgres.md)

Configure Galaxy to use Postgres - edit `~/galaxy-dist/universe_wsgi.ini` and uncomment the `database_connection` configuration. Modify it for your postgres settings. This should be something like:

    postgres:///galaxy?user=galaxy&password=galaxy
    
Install and configure Apache

    sudo yum install httpd httpd-devel
    sudo vi /etc/httpd/conf.d/galaxy.conf

and add the following (adjusting paths as needed):

    RewriteEngine on
    RewriteRule ^/static/style/(.*) /home/galaxy/galaxy-dist/static/june_2007_style/blue/$1 [L]
    RewriteRule ^/static/scripts/(.*) /home/galaxy/galaxy-dist/static/scripts/packed/$1 [L]
    RewriteRule ^/static/(.*) /home/galaxy/galaxy-dist/static/$1 [L]
    RewriteRule ^/favicon.ico /home/galaxy/galaxy-dist/static/favicon.ico [L]
    RewriteRule ^/robots.txt /home/galaxy/galaxy-dist/static/robots.txt [L]
    RewriteRule ^(.*) http://localhost:8080$1 [P]

Restart Galaxy    

    ./galaxy stop
    ./galaxy start
    
Restart Apache    

    sudo chkconfig --level 345 httpd on
    sudo service httpd restart


### Smoke Test
You can verify that your install has been successful by executing these tests:

TODO
    
