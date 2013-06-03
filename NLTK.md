### Background
NLTK is a Python library used for text analysis (lexical anaysis, tokenisation and parsing). It is integrated into Galaxy for HCSvLab.
* [NLTK Home Page](http://nltk.org/) 
* [Source Code](https://github.com/nltk/nltk) 

### Assumptions
* You have a CentOS machine. 
* You have a non-root user account on this machine with sudo privileges. 
* You are logged in as this user and are in the home directory.
* You have python installed. Type the following in your terminal to see your python install version:

    python -V    


### Setup
Install python tools
 
    sudo yum update  #IS THIS NEEDED?
    sudo yum install python-devel python-setuptools

Install Pip

    sudo easy_install pip

Install Numpy

    sudo pip install -U numpy
    
Install PyYAML and NLTK

    sudo easy_install -U distribute 
    sudo pip install -U pyyaml nltk
    
    sudo mkdir /usr/share/nltk_data
    sudo python -m nltk.downloader -d /usr/share/nltk_data all


