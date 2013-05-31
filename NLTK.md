### Assumptions:
You have a fresh CentOS machine. You have a user account on this machine with sudo privileges. You are logged in as this user and are in the home directory.

You have python installed. Type the following in your terminal to see your python install version:

    python -v    

### Setup
Setup some python tools
 
    sudo yum update
    sudo yum install python-devel

Install Pip

    sudo easy_install pip

Install Numpy

    sudo pip install -U numpy
    
Install PyYAML and NLTK

    sudo easy_install distribute 
    sudo pip install -U pyyaml nltk
    
    sudo mkdir /usr/share/nltk_data
    sudo python -m nltk.downloader -d /usr/share/nltk_data all
