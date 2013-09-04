### Developer Setup - Web App
### Install PhantomJS
For more info, visit: http://phantomjs.org/download.html

**In Ubuntu**

    $ cd /usr/local/share/
    $ sudo wget https://phantomjs.googlecode.com/files/phantomjs-1.9.1-linux-x86_64.tar.bz2
    $ sudo tar jxvf phantomjs-1.9.1-linux-x86_64.tar.bz2
    $ sudo ln -s /usr/local/share/phantomjs-1.9.1-linux-x86_64/ /usr/local/share/phantomjs
    $ sudo ln -s /usr/local/share/phantomjs/bin/phantomjs /usr/local/bin/phantomjs
    
**In Mac OS X**

    brew update && brew install phantomjs
    
Unfortunately, installing with MacPorts is not recommended.

### Install Postgres
**In Ubuntu**

    $ sudo apt-get install postgresql libpq-dev # install postgresql and libpq-dev packages
    $ sudo -u postgres psql template1 # login to template1 database
    # \password postgres # change password for user postgres
    # \q # exit	

###Create Database User

    $ sudo -u postgres createuser -P # create a new user called hcsvlab (enable create databases) with password hcsvlab

###Clone HCSVLab

    $ rvm install ruby-2.0.0-p0
    $ rvm use ruby-2.0.0-p0@hcsvlab \--create
    $ git clone git@github.com:IntersectAustralia/hcsvlab.git

###Install ActiveMQ
**In Ubuntu**

    $ curl http://mirror.mel.bkb.net.au/pub/apache/activemq/apache-activemq/5.8.0/apache-activemq-5.8.0-bin.tar.gz | tar xvz
 
***#set configuration, using provided file from HCSVLab project***

    $ cd apache-activemq-5.8.0
    $ cp <hcsvlab folder>/activemq_conf/activemq.xml conf/activemq.xml
    $ bin/activemq start

###Setup HCSVLAB

    $ cd ~/hcsvlab
    $ git submodule init
    $ git submodule update
    $ bundle install
    $ rake db:create db:migrate db:seed db:populate
    $ rake jetty:config
    $ rake jetty:start
    $ rake a13g:start_pollers
    $ rails s

###Ingest an AusNC Corpus into Fedora
For example data, see Download the Corpus in Installing AusNC.

After that, refer to https://github.com/IntersectAustralia/hcsvlab_robochef#installation for more instructions.

    $ rake fedora:ingest corpus=/path/to/corpus/directory

###Ingest a single AusNC Corpus Item into Fedora

    $ rake fedora:ingest_one /path/to/corpus/rdf/file

###Clear all corpora from Fedora

    $ rake fedora:clear
