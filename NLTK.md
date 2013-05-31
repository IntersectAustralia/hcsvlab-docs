### About NLTK
NLTK is a Python library used for text analysis (lexical anaysis, tokenisation and parsing). 

[NLTK Home Page](http://nltk.org/) 

[Source Code](https://github.com/nltk/nltk) 

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

### To use NLTK

Run python

    python
    
within python

    import nltk


### Import text from a URL

    from urllib import urlopen
    url = "ENTER_URL_HERE"
    raw = urlopen(url).read()
    raw

###Load plaintext corpora from a local directory
    
    from nltk.corpus import PlaintextCorpusReader
    
Import corpus data with the following command.
The first argument is the path to the folder containing the text files.
The second argument is a regular expression for selecting the files within that folder. 

    corpus = PlaintextCorpusReader('./some_corpus', '.*')

Corpus objects have methods for returning raw text, tokens, sentences, and paragraphs from the entire corpus

        
    corpus.raw() # return the original plain text
    corpus.words() # returns an array of word tokens
    corpus.sents() # returns an array of word arrays representing sentences
    corpus.paras() # returns an array of sentence arrays
    
    

The above commands can be applied to a specific file within the corpus

    
    > corpus.fileids() # list all text files within the corpus
        ['s1a-006-plain.txt', 's1a-007-plain.txt', 's1a-008-plain.txt']
    > corpus.words('s1a-008-plain.txt')
        ['Um', 'about', 'three', 'days', 'ago', 'we', 'saw', ...]
        
###Convert text to sentences

    sentence_tokenizer = nltk.data.load('tokenizers/punkt/english.pickle')
    text = "PUT YOUR TEXT HERE" 
    sentences = sentence_tokenizer.tokenize(text)
    print sentences 

###Produce Part-of-Speech Tags

    text = nltk.word_tokenize("YOUR_TEXT_HERE")
    nltk.pos_tag(text)
