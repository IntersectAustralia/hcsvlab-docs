### Background

The JC-NLTK Wrapper is a tool for integrating the Johnson Charniak Parser into NLTK.

### Assumptions:
* You have a CentOS machine. 
* You have a non-root user account on this machine with sudo privileges. 
* You are logged in as this user and are in the home directory.
* You have already installed [NLTK](NLTK.md) and [The Johnson Charniak Parser](JCParser.md)


### Setup

Install the python library `TKinter`

    sudo yum install tkinter

Clone the repository

    cd ~
    git clone git://github.com/IntersectAustralia/jcp-nltk-wrapper.git
    
Now you need to find where NLTK is installed. On Centos this is likely to be something like `/usr/lib/python2.6/site-packages/nltk/parse` - adjust paths below if yours is different.    

Copy the files under `~/jcp-nltk-wrapper/parse` to the python NLTK installation directory as per above

    sudo cp ~/jcp-nltk-wrapper/parse/* /usr/lib/python2.6/site-packages/nltk/parse
    
Edit `/usr/lib/python2.6/site-packages/nltk/parse/johnsoncharniak.ini` to point your installation of the Johnson Charniak parser, which should be ~/reranking-parser

    basedir: /home/galaxy/reranking-parser
    
Edit `/usr/lib/python2.6/site-packages/nltk/parse/johnsoncharniak.py` so that the `config.read` line points your edited copy of johnsoncharniak.ini

    config.read('/usr/lib/python2.6/site-packages/nltk/parse/johnsoncharniak.ini')

