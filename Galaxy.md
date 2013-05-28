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

Install Galaxy customisations and tool wrappers

    cd ~/galaxy-dist
    git init
    git remote add origin git@github.com:IntersectAustralia/hcsvlab-galaxy.git
    git pull origin master
    chmod 755 galaxy
    ./galaxy start

    