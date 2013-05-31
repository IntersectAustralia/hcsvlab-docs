### Assumptions:
You have a fresh CentOS machine. You have a user account on this machine with sudo privileges. You are logged in as this user and are in the home directory.

### Setup
Set up dev tools

    sudo yum groupinstall "Development Tools"

Install mercurial if you dont have it

    sudo yum install '*mercurial*'

Pull down and install the default galaxy distribution

    hg clone https://bitbucket.org/galaxy/galaxy-dist/
    cd galaxy-dist
    hg update stable

[Install NLTK](NLTK.md) 

[Install the Johnson Charniak Parser](JCParser.md)

[Install the JC Parser NLTK wrapper](JCPNLTKWrapper.md)

Install the following required tools (for displaying parse trees)
    
    sudo yum install tkinter
    sudo yum install python-matplotlib-tk.x86_64
    sudo yum install xorg-x11-server-Xvfb
    sudo yum install ImageMagick
    Xvfb :1 -screen 0 1024x768x24 &
    export DISPLAY=:1
    sudo yum install xhost
    xhost +
    
Add the `DISPLAY` configuration to your bashrc

    cd ~
    nano .bashrc
        // Paste the following line into the file:
        export DISPLAY=:1
    source ~/.bashrc

Install Galaxy customisations and tool wrappers

    cd ~/galaxy-dist
    git init
    git remote add origin git@github.com:IntersectAustralia/hcsvlab-galaxy.git
    git pull origin master
    chmod 755 galaxy
    ./galaxy start
    
<b> Note: </b> If setting up locally, or there is some issue with the .galaxy start command, nano into galaxy file and check the path to galaxy directory and the user is correct    

    
