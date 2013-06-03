### Background
These instructions are to install Galaxy and the HCSvLab tools on a CentOS machine. This can easily be adapted to other distributions.

The Galaxy instructions are based on the following from the Galaxy website
* [Basic install](http://wiki.galaxyproject.org/Admin/Get%20Galaxy) 
* [Running in production](http://wiki.galaxyproject.org/Admin/Config/Performance/ProductionServer)

### Assumptions
* You have a fresh CentOS machine. 
* You have a non-root user account on this machine with sudo privileges. 
* You are logged in as this user and are in the home directory.

### Setup
Install development tools

    sudo yum groupinstall "Development Tools"

Install mercurial if you dont have it

    sudo yum install mercurial

Pull down and install the default galaxy distribution

    hg clone https://bitbucket.org/galaxy/galaxy-dist/
    cd galaxy-dist
    hg update stable

[Install NLTK](NLTK.md) 

[Install the Johnson Charniak Parser](JCParser.md)

[Install the JC Parser NLTK wrapper](JCPNLTKWrapper.md)

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

Install Galaxy customisations and tool wrappers

    cd ~/galaxy-dist
    git init
    git remote add origin git://github.com/IntersectAustralia/hcsvlab-galaxy.git
    git pull origin master
    chmod 755 galaxy
    ./galaxy start   #GIVES ERROR WHEN NOT devel *** ERROR *** must be devel or root in order to control this service
    
TODO
* Postgres
* Apache
* Run as a service
* Smoke test with tools

    
