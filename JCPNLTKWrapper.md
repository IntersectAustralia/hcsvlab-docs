### About JC-NLTK Wrapper

The JC-NLTK Wrapper is a tool for integrating the Johnson Charniak Parser into NLTK.

### Assumptions:
* You have a fresh CentOS machine. 
* You have a non-root user account on this machine with sudo privileges. 
* You are logged in as this user and are in the home directory.
* You have already installed [NLTK](NLTK.md) and [The Johnson Charniak Parser](JCParser.md)


### Setup

Install the python library `TKinter`
 
    sudo yum install tkinter

Clone the repository

    cd ~
    git clone git://github.com/IntersectAustralia/jcp-nltk-wrapper.git
    
Copy the files under `jcp-nltk-wrapper/parse` to the python NLTK installation

    cp jcp-nltk-wrapper/parse/* /usr/local/lib/python2.7/dist-packages/nltk/parse 
    
Edit `parse/johnsoncharniak.ini` (in the python NLTK directory) to point your installation of the Johnson Charniak parser

    >> basedir: 'your/jc_parser_directory'
    
Edit `parse/johnsoncharniak.py` (in the python NLTK directory) so that the `config.read` line in points your edited copy of johnsoncharniak.ini

    >> config.read('your_python_nltk_directory/parse/johnsoncharniak.ini')

