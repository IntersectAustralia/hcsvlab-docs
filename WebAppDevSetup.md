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

**In Mac OS X**

MAC OS X often comes with its own postgres, so ensure any other version is stopped or uninstalled

    $ brew install postgres
    $ initdb /usr/local/var/postgres -E utf8
    $ postgres -D /usr/local/var/postgres

###Create Database User
**In Ubuntu**

    $ sudo -u postgres createuser -P # create a new user called hcsvlab (enable create databases) with password hcsvlab

**In Mac OS X**

    $ sudo -u <home_user> createuser hcsvlab # with password hcsvlab
    $ psql
    $ ALTER USER hcsvlab CREATEDB;
    $ \q

###Clone HCSVLab
**In Ubuntu and Mac OS X**

    $ rvm install ruby-2.0.0-p0
    $ rvm use ruby-2.0.0-p0@hcsvlab \--create
    $ git clone git@github.com:IntersectAustralia/hcsvlab.git
    
###Install ImageMagick (required for the rmagick gem)
**In Ubuntu**

    $ sudo apt-get install imagemagick

**In Mac OS X**

    $ brew install imagemagick

###Install ActiveMQ
**In Ubuntu and Mac OS X**

    $ curl http://mirror.mel.bkb.net.au/pub/apache/activemq/apache-activemq/5.8.0/apache-activemq-5.8.0-bin.tar.gz | tar xvz
 
***#set configuration, using provided file from HCSVLab project***

    $ cd apache-activemq-5.8.0
    $ cp <hcsvlab folder>/activemq_conf/activemq.xml conf/activemq.xml
    $ bin/activemq start

###Setup HCSVLAB
**For Ubuntu and Mac OS X**

    $ cd ~/hcsvlab
    $ git submodule init
    $ git submodule update
    $ bundle install
    $ rake db:create db:migrate db:seed db:populate
    $ rake jetty:config
    $ rake jetty:start
    $ rake a13g:start_pollers
    $ rails s
    
###Setup https for AAF
**For Mac OS X**

For testing purpose, please register your application [here](https://rapid.test.aaf.edu.au/registration), it will ask you to login through aaf. Please select idptest.intersect.org.au

Once you are logged in, please fill in the details

<table>
<tr>
<th>Field</th>
<th>Value</th>
</tr>
<tr>
<td>Organisation</td>
<td>Intersect_test</td>
</tr>
<tr>
<td>Name</td>
<td>Your application name</td>
</tr>
<tr>
<td>URL</td>
<td>Fake application URL e.g. https://hcsvlab.intersect.org.au</td>
</tr>
<tr>
<td>Callback URL</td>
<td>The same URL but append /users/aaf_sign_in to the URL. e.g. https://hcsvlab.intersect.org.au/users/aaf_sign_in</td>
</tr>
<tr>
<td>Secret</td>
<td>Please generate by following command: 
    <code>tr -dc '[[:alnum:][:punct:]]' &lt; /dev/urandom | head -c32 ;echo </code>
</td>
</tr>
</table>

Please remember the secret token. Once you submit this, you will get a url. Please also keep the url to be used in your configuration.

**Configure apache2**

This has been tested on OSX 10.9.
Add a new line to the /etc/hosts

    127.0.0.1       hcsvlab.intersect.org.au
    
Please create the certificate for the ssl, answer all questions

    sudo mkdir -p /etc/ssl/private
    sudo mkdir /etc/ssl/certs
    sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/hcsvlab.key -out /etc/ssl/certs/hcsvlab.crt

Create a new file in /etc/apache2/ naming it hcsvlab.conf and put the content:

    <VirtualHost *:80>
        ServerName hcsvlab.intersect.org.au
        Redirect permanent / https://hcsvlab.intersect.org.au/
    </VirtualHost>
    <VirtualHost _default_:443>
        ServerName hcsvlab.intersect.org.au
        SSLEngine on
        ErrorLog /var/log/apache2/ssl_error_log
        CustomLog /var/log/apache2/ssl_access_log combined
        CustomLog /var/log/apache2/ssl_request_log \
              "%t %h %{SSL_PROTOCOL}x %{SSL_CIPHER}x 636f4ebbf2a79889b02804364ae277836ade24bfquot;%r636f4ebbf2a79889b02804364ae277836ade24bfquot; %b"
        LogLevel warn
        SetEnvIf User-Agent ".*MSIE.*" \
             nokeepalive ssl-unclean-shutdown \
             downgrade-1.0 force-response-1.0
        SSLProtocol all -SSLv2
        SSLCipherSuite ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP:+eNULL
        SSLCertificateFile /etc/ssl/certs/hcsvlab.crt
        SSLCertificateKeyFile /etc/ssl/private/hcsvlab.key
     
        ProxyPreserveHost On
        ProxyPass / http://localhost:3000/
        ProxyPassReverse / http://localhost:3000/
        ProxyRequests On
        <Proxy *>
          Order deny,allow
          Allow from all
        </Proxy>
    </VirtualHost>

Then restart apace
    
    sudo apachectl -k restart


###Setup HCSVLAB cucumber test setup
**For Ubuntu and Mac OS X**

    $ RAILS_ENV=test bundle exec rake db:create db:migrate db:test:prepare

###Ingest an AusNC Corpus into Fedora
For example data, see Download the Corpus in Installing AusNC.

After that, refer to https://github.com/IntersectAustralia/hcsvlab_robochef#installation for more instructions.

    $ rake fedora:ingest corpus=/path/to/corpus/directory

###Ingest a single AusNC Corpus Item into Fedora

    $ rake fedora:ingest_one /path/to/corpus/rdf/file

###Clear all corpora from Fedora

    $ rake fedora:clear
