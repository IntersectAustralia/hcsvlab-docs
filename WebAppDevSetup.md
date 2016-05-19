### Developer Setup - Virtual Environment

The HCSVLAB web app developer installation instructions assume an OSX or Ubuntu environment. To set up an Ubuntu virtual machine run through the following steps. If planning to use an OSX environment ignore this section and continue at the Developer Setup - Web App section.

**Install Virtualbox**

Virtualbox will be used to set up virtual machines on your computer. It can be found at https://www.virtualbox.org/
A user manual containing installation instructions can be found at https://www.virtualbox.org/manual/UserManual.html however installation should be as simple as downloading and clicking the installer.

**Obtain an Ubuntu image**

An Ubuntu image will need to be provided to virtual box for it to be able to install Ubuntu onto a virtual machine. Either the desktop version http://www.ubuntu.com/download/desktop or server version http://www.ubuntu.com/download/server can be used.

**Create a new Virtual Machine**

Once the Ubuntu image is downloaded open Virtualbox and click to create a new vm. Give it some appropriate name such as HCSVLab, set its type to "Linux" and version to "Ubuntu (64-bit)". Allocate the VM 1024 mb of RAM, and use the default options to create a dynamically allocated physical hard disk.

Configure the settings of the VM within Virtualbox to have two network adapters. The first should be a bridged adapter and the second a Host-only adapter. This will allow the vm to access the internet and allow ssh'ing into the vm from the host machine.

To make development easier it is handy to clone the HCSVLAB repository onto the host machine (desktop) and set up a shared drive to the cloned repo. This allows code changes to be made to the cloned repo within the desktop and still have the changes take effect within the vm. When creating the shared drive ensure to enable the settings "Make permanent" and "Auto mount".

Once the virtual machine has been created start it up and select the downloaded Ubuntu image file when prompted for a virtual disk. Run through the Ubuntu setup, making sure to give the user a memorable name and password ("devel" is normally used as it aligns with our server environments). Once setup is complete and you have logged in install guest additions onto the vm by clicking "Devices > Insert Guest Additions CD Image" within the Virtualbox menu toolbar and follow the instructions within the VM. Once guest additions are installed add your Ubuntu user to the vboxsf group so that they can access the shared folder:

    $ sudo usermod -aG vboxsf $(whoami)
    
After logging out and in again you should be able to access the shared folder at "/media/sf_<shared folder name>". Setup of the Ubuntu virtual machine should now be complete. 

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

The easiest way to set up Postgres on Mac OS X is to download from http://postgresapp.com/ and follow the "Installing Postgres.app instructions at http://postgresapp.com/documentation/install.html.

***After installation***
1. Run `echo 'export PATH=$PATH:/Applications/Postgres.app/Contents/Versions/9.3/bin' > ~/.bash_profile` in the terminal. This will give you access to the command line tools.
2. Open Postgres.app from Applications. You should see an elephant icon on the menu bar.
3. Left click on the elephant icon and go the Preferences
4. Uncheck "Show Welcome Window..."
5. Check "Start Postgres automatically after login"
    
###Create Database User
**In Ubuntu**

    $ sudo -u postgres createuser -P # create a new user called hcsvlab (enable create databases) with password hcsvlab

If you already have a "postgres" user you may get the following error `$ createuser: creation of new role failed: ERROR:  role "postgres" already exists`. This means that a new user with the name hcsvlab won't be created which can cause the database rake in later steps to result in an error. To resolve this you need to open psql as the postgres user and add a new role with the name "hcsvlab" and the relevant permissions.

    $ sudo su - postgres # log into postgres user
    $ psql # open psql console
    # CREATE ROLE hcsvlab WITH LOGIN CREATEDB;
    # \du
    # \q
    $ exit

**In Mac OS X**

    $ createuser -sP hcsvlab # use hcsvlab as password

###Clone HCSVLab
If Ruby Version Manager is not already installed on the development environment follow the instructions at https://rvm.io/rvm/install

**In Ubuntu and Mac OS X**

    $ rvm install ruby-2.1.4
    $ rvm use ruby-2.1.4@hcsvlab \--create
    $ gem install bundler
    $ git clone git@github.com:IntersectAustralia/hcsvlab.git
    
###Install ImageMagick (required for the rmagick gem)
**In Ubuntu**

    $ sudo apt-get install imagemagick

**In Mac OS X**

    $ brew install imagemagick

###Install ActiveMQ
**In Ubuntu and Mac OS X**

    $ curl http://archive.apache.org/dist/activemq/apache-activemq/5.8.0/apache-activemq-5.8.0-bin.tar.gz | tar xvz
 
***#set configuration, using provided file from HCSVLab project***

    $ cd apache-activemq-5.8.0
    $ cp <hcsvlab folder>/activemq_conf/activemq.xml conf/activemq.xml
    $ bin/activemq start

If running `$ bin/activemq start` in Ubuntu results in an error message `ERROR: Configuration variable JAVA_HOME or JAVACMD is not defined correctly.`, then it is likely that no Java Runtime Environment is installed. If `$ java -version` doesn't list an installed java version, then install the latest version of the JRE using `$ sudo apt-get install default-jre`.

###Setup HCSVLAB
**For Ubuntu and Mac OS X**

    $ cd ~/hcsvlab
    $ git submodule init
    $ git submodule update
    $ bundle install
    $ rake db:create db:migrate db:seed db:populate
    $ rake jetty:reset_all # only run this command the first time for subsequent starts run $ rake jetty:start a13g:start_pollers
    $ rails s

To check that the required processes are running, the script `/hcsvlab/script/system_check.sh` can be run.
    
If running `$ bundle install` results in the following error, then try installing the bundler gem `$ gem install bundler`.

    .../kernel_require.rb:54:in `require': cannot load such file -- bundler (LoadError)

###Setup Data Directory
**For Ubuntu and Mac OS X**

    $ mkdir -p /data/contributed_annotations
    $ chown -R <user>:<user> /data
    
###Run Tests
**For Ubuntu and Mac OS X**

    $ cd ~/hcsvlab
    $ RAILS_ENV=test rake db:drop db:create db:migrate
    $ rspec spec
    $ cucumber features
    
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

Add the following lines to `/etc/apache2/httpd.conf`

    Listen 443
    ServerName hcsvlab.intersect.org.au
    Include /etc/apache2/hcsvlab.conf

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
