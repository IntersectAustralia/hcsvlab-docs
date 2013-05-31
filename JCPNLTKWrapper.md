### About JC-NLTK Wrapper

The JC-NLTK Wrapper is a tool for integrating the Johnson Charniak Parser into NLTK.

### Assumptions:
You have a fresh CentOS machine. You have a user account on this machine with sudo privileges. You are logged in as this user and are in the home directory.

You have already installed:
[NLTK](NLTK.md) and
[The JC-Parser](JCParser.md)


### Setup

Install the  python library `TKinter`
 
    sudo yum install tkinter

Clone the repository

    git clone git@github.com:IntersectAustralia/jcp-nltk-wrapper.git
    
Copy the files under `jcp-nltk-wrapper/parse` to the python NLTK installation

    cp jcp-nltk-wrapper/parse/* /usr/local/lib/python2.7/dist-packages/nltk/parse 
    #or
    cp jcp-nltk-wrapper/parse/* /Library/Python/2.7/site-packages/nltk/parse    
    
Edit `parse/johnsoncharniak.ini` (in the python NLTK directory) to point the your installation of the Johnson Charniak parser

    >> basedir: 'your/jc_parser_directory'
    
Edit `parse/johnsoncharniak.py` (in the python NLTK directory) so that the `config.read` line in points your edited copy of johnsoncharniak.ini

    >> config.read('your_python_nltk_directory/parse/johnsoncharniak.ini')

### Run the Wrapper

Run python

    python

Import libraries

    import nltk
    import TKinter
    
The parse() method takes a sentence as input, which in NLTK is usually an array of string tokens

    sent = ['the', 'cat', 'sat', 'on', 'the', 'mat']

Run the parser

    tree = parser.parse(sent)
    
Check the output

    tree.draw()
