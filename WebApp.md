### Background

These instructions describe the process of installing and configuring the HCSvLab web application and it components on a CentOS machine

### Assumptions
* You have a fresh CentOS machine. 
* You have a non-root user account on this machine with sudo privileges. We used "devel".
* You are logged in as this user and are in the home directory.

### Setup

**Correct directory permissions**

    $ sudo chmod og+rx /home/devel

**Turn off SELinux**

    $ sudo setenforce 0
    
    $ sudo vi /etc/selinux/config
    ...
    SELINUX=disabled  # Change SELINUX value to disabled

**Install EPEL**

These are additional repositories that are not enabled by default but contain some of the packages required by the HCSvLab web application

    $ curl -O http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
    $ curl -O  http://rpms.famillecollet.com/enterprise/remi-release-6.rpm
    $ sudo rpm -Uvh remi-release-6*.rpm epel-release-6*.rpm

**Install git and other useful packages**

    $ sudo yum install gcc gcc-c++ patch readline readline-devel zlib zlib-devel libyaml-devel libffi-devel openssl openssl-devel make bzip2 autoconf automake libtool bison httpd httpd-devel apr-devel apr-util-devel mod_xsendfile ntp postgresql postgresql-libs postgresql-server postgresql-devel postfix curl curl-devel openssl openssl-devel tzdata libxml2 libxml2-devel libxslt libxslt-devel sqlite-devel git system-config-firewall-tui ImageMagick ImageMagick-devel

**Make directories to rotate Apache log files**

    $ sudo mkdir /var/log/httpd/old
    $ sudo chmod 700 /var/log/httpd/old

**Setup Postgress**

Create a "hcsvlab" user and database in [Postgres](Postgres.md)

**Install RVM and Ruby**

RVM is a Ruby Version Manager, for more information see [rvm.io](http://rvm.io)

    $ \curl -#L https://get.rvm.io | bash -s stable --autolibs=3 --ruby=2.0.0-p0
    $ source /home/devel/.rvm/scripts/rvm
    $ rvm gemset create hcsvlab # this is the gemset cap deploy will use

**Install Passenger**

Passenger is an Apache2 module that serves Ruby on Rails applications. For more information see [Phusion Passanger](http://www.modrails.com/)

Install

    $ gem install passenger
    $ passenger-install-apache2-module
    
Configure

> Note: The values for serverName and any file paths should specify values valid for your environment
	
    $ sudo vi /etc/httpd/conf.d/hcsvlab.conf
    ...
    <VirtualHost *:80>
      ServerName ic2-hcsvlab-qa2-vm.intersect.org.au
      DocumentRoot /home/devel/hcsvlab-web/current/public
      LoadModule passenger_module /home/devel/.rvm/gems/ruby-2.0.0-p0/gems/passenger-4.0.5/libout/apache2/mod_passenger.so
      PassengerRoot /home/devel/.rvm/gems/ruby-2.0.0-p0/gems/passenger-4.0.5
      PassengerDefaultRuby /home/devel/.rvm/wrappers/ruby-2.0.0-p0/ruby
      RailsEnv production
      # Uploads of up to 100MB permitted
      LimitRequestBody 100000000
      <Directory /home/devel/hcsvlab-web/current/public>
        AllowOverride all
        Options -MultiViews
      </Directory>
    </VirtualHost>
    ...

**Open Ports for the Web Services**

Edit `iptables` to open up port 80, and optionally 8080 for Tomcat (see below):

    $ sudo vi /etc/sysconfig/iptables
	
Add the lines:

    -A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT
    -A INPUT -m state --state NEW -m tcp -p tcp --dport 8080 -j ACCEPT

right after this line:

    -A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT
    
Restart the service:

    $ sudo service iptables restart
	
Restart Apache -- won't work without Passenger installed

    $ sudo chkconfig --level 345 httpd on
    $ sudo service httpd restart

**Make a directory for the app (capistrano needs this)**

    $ mkdir ~/hcsvlab-web
    $ mkdir ~/hcsvlab-web/releases

**Install Java**

Download JDK 6 update 45 from Oracle.

[jdk-6u45-linux-x64-rpm.bin](http://www.oracle.com/technetwork/java/javase/downloads/jdk6downloads-1902814.html)

This requires clicking a license agreement, so you will have to download it to your local machine, then scp it to your target machine.

    $ scp jdk-6u45-linux-x64-rpm.bin devel@ic2-hcsvlab-qa2-vm.intersect.org.au:~/downloads
    $ ssh devel@ic2-hcsvlab-qa2-vm.intersect.org.au
    ...
    $ cd downloads
    $ chmod +x jdk-6u45-linux-x64-rpm.bin
    $ sudo ./jdk-6u45-linux-x64-rpm.bin
    $ sudo alternatives --install /usr/bin/java java /usr/java/jdk1.6.0_45/jre/bin/java 20000
    $ sudo alternatives --install /usr/bin/javac javac /usr/java/jdk1.6.0_45/bin/javac 20000
    $ sudo alternatives --install /usr/bin/jar jar /usr/java/jdk1.6.0_45/bin/jar 20000
    $ sudo alternatives --config java
    $ sudo alternatives --config javac
    $ sudo alternatives --config jar
	
**Install ActiveMQ**

ActiveMQ is the messaging component used by the system. For more informaiton see the [Apache ActiveMQ](http://activemq.apache.org/) site.

    $ wget http://mirror.ventraip.net.au/apache/activemq/apache-activemq/5.8.0/apache-activemq-5.8.0-bin.tar.gz
    $ tar -xvzf apache-activemq-5.8.0-bin.tar.gz
    $ sudo mv apache-activemq-5.8.0 /opt
    $ sudo ln -s /opt/apache-activemq-5.8.0 /opt/activemq
    $ sudo chown -R devel:devel /opt/activemq
	
**Install Tomcat**

Tomcat is the serverlet container that runs SOLR and Fedora Commons. Form more information see the [Apache Tomcat](http://tomcat.apache.org/) site.

    $ curl -O http://apache.mirror.serversaustralia.com.au/tomcat/tomcat-6/v6.0.37/bin/apache-tomcat-6.0.37.tar.gz
    $ tar -xvzf apache-tomcat-6.0.37.tar.gz
    $ sudo mv apache-tomcat-6.0.37 /opt
    $ sudo ln -s /opt/apache-tomcat-6.0.37 /opt/tomcat
    $ sudo chown -R devel:devel /opt/tomcat
	
**Install Solr**

Solr is the search engine platform used by the web application, for more information see the [Apache Solr](http://lucene.apache.org/solr/) site.

Solr is installed during project deployment, but we must make a directory structure for it.

    $ sudo mkdir -p /opt/solr/hcsvlab/solr/hcsvlab-core/conf
    $ sudo chown -R devel:devel /opt/solr

**Install Fedora Commons**

Fedora is the repository manager that handles metadata and persistance for the web application, for more information see the [Fedora Commons](http://fedora-commons.org/) site.

Download the installer and create a place for it to install to:

    $ curl -OL http://sourceforge.net/projects/fedora-commons/files/fedora/3.6.1/fcrepo-installer-3.6.1.jar
    $ sudo mkdir /opt/fedora
    $ sudo chown -R devel:devel /opt/fedora
    
Run the installer:

    $ java -jar fcrepo-installer-3.6.1.jar
    
Follow the prompts, selection the same options as indicated below:

> NOTE: Values for server names and file paths should specify values valid for your environment

    ***********************
	  Fedora Installation
	***********************
	
	To install Fedora, please answer the following questions.
	Enter CANCEL at any time to abort the installation.
	Detailed installation instructions are available online:
	
	            https://wiki.duraspace.org/display/FEDORA/All+Documentation
	
	Installation type
	-----------------
	The 'quick' install is designed to get you up and running with Fedora
	as quickly and easily as possible. It will install Tomcat and an
	embedded version of the Derby database. SSL support and XACML policy
	enforcement will be disabled.
	For more options, including the choice of hostname, ports, security,
	and databases, select 'custom'.
	To install only the Fedora client software, enter 'client'.
	
	Options : quick, custom, client
	
	Enter a value ==> custom
	
	
	Fedora home directory
	---------------------
	This is the base directory for Fedora scripts, configuration files, etc.
	Enter the full path where you want to install these files.
	
	Enter a value ==> /opt/fedora
	
	WARNING: The environment variable, FEDORA_HOME, is not defined
	WARNING: Remember to define the FEDORA_HOME environment variable
	WARNING: before starting Fedora.
	
	Fedora administrator password
	-----------------------------
	Enter the password to use for the Fedora administrator (fedoraAdmin) account.
	
	Enter a value ==> fedoraAdmin
	
	
	Fedora server host
	------------------
	The host Fedora will be running on.
	If a hostname (e.g. www.example.com) is supplied, a lookup will be
	performed and the IP address of the host (not the host name) will be used
	in the default Fedora XACML policies.
	
	
	Enter a value [default is localhost] ==> ic2-hcsvlab-qa2-vm.intersect.org.au
	
	
	Fedora application server context
	---------------------------------
	The application server context Fedora will be running in.
	If 'fedora' (default) is supplied, the resulting context path
	will be http://www.example.com/fedora.
	It must be ensured that the configured application server context
	matches this path if explicitly configured.
	
	
	Enter a value [default is fedora] ==>
	
	
	Authentication requirement for API-A
	------------------------------------
	Fedora's management (API-M) interface always requires user authentication.
	Require user authentication for Fedora's access (API-A) interface?
	
	Options : true, false
	
	Enter a value [default is false] ==>
	
	
	SSL availability
	----------------
	Should Fedora be available via SSL?  Note: this does not preclude
	regular HTTP access; it just indicates that it should be possible for
	Fedora to be accessed over SSL.
	
	Options : true, false
	
	Enter a value [default is true] ==>
	
	
	SSL required for API-A
	----------------------
	Should API-A be accessible exclusively via SSL?  If true, requests
	to access API-A URLs will be automatically redirected to the secure port.
	
	Options : true, false
	
	Enter a value [default is false] ==>
	
	
	SSL required for API-M
	----------------------
	Should API-M be accessible exclusively via SSL?  If true, requests
	to access API-M URLs will be automatically redirected to the secure port.
	
	Options : true, false
	
	Enter a value [default is true] ==>
	
	
	Servlet engine
	--------------
	Which servlet engine will Fedora be running in?
	Enter 'included' to use the bundled Tomcat 6.0.35 server.
	To use your own, existing installation of Tomcat, enter 'existingTomcat'.
	Enter 'other' to use a different servlet container.
	
	Options : included, existingTomcat, other
	
	Enter a value [default is included] ==> existingTomcat
	
	
	Tomcat home directory
	---------------------
	Please provide the full path to your existing Tomcat installation, or
	the path where you plan to install the bundled Tomcat.
	
	Enter a value ==> /opt/apache-tomcat-6.0.37
	
	WARNING: The environment variable, CATALINA_HOME, is not defined
	WARNING: Remember to define the CATALINA_HOME environment variable
	WARNING: before starting Fedora.
	
	Tomcat HTTP port
	----------------
	Which HTTP port (non-SSL) should Tomcat listen on?  This can be changed
	later in Tomcat's server.xml file.
	
	
	Enter a value [default is 8080] ==>
	
	
	Tomcat shutdown port
	--------------------
	Which port should Tomcat use for shutting down?  Make sure this doesn't
	conflict with an existing service.  This can be changed later in Tomcat's
	server.xml file.
	
	
	Enter a value [default is 8005] ==>
	
	
	Tomcat Secure HTTP port
	-----------------------
	Which port (SSL) should Tomcat listen on?  This can be changed
	later in Tomcat's server.xml file.
	
	
	Enter a value [default is 8443] ==>
	
	
	Keystore file
	-------------
	For SSL support, Tomcat requires a keystore file.
	If the keystore file is located in the default location expected by
	Tomcat (a file named .keystore in the user home directory under which
	Tomcat is running), enter 'default'.
	Otherwise, please enter the full path to your keystore file, or, enter
	'included' to  use the the  sample, self-signed certificate) provided by
	the installer.
	For more information about the keystore file, please consult:
	http://tomcat.apache.org/tomcat-6.0-doc/ssl-howto.html.
	
	Enter a value ==> included
	
	
	Database
	--------
	Please select the database you will be using with
	Fedora. The supported databases are Derby, MySQL, Oracle and Postgres.
	If you do not have a database ready for use by Fedora or would prefer to
	use the embedded version of Derby bundled with Fedora, enter 'included'.
	
	Options : derby, mysql, oracle, postgresql, included
	
	Enter a value ==> included
	
	
	Use upstream HTTP authentication (Experimental Feature)
	-------------------------------------------------------
	You may wish to rely on a local SSO or other external source for HTTP
	authentication and subject attributes.
	WARNING: This is an experimental feature and should be enabled only with the
	understanding that integration with external authentication will require
	further configuration and that this is not yet a stable Fedora feature.
	We invite you to try it out and give us feedback.
	Use upstream authentication?
	
	Options : true, false
	
	Enter a value [default is false] ==>
	
	
	Enable FeSL AuthZ (Experimental Feature)
	----------------------------------------
	Enable FeSL Authorization? This is an experimental replacement for Fedora's
	legacy authorization module, and is still under development.
	Production repositories should NOT enable this, but we invite you to try it
	out and give us feedback.
	
	
	Enter a value [default is false] ==>
	
	
	Policy enforcement enabled
	--------------------------
	Should XACML policy enforcement be enabled?  Note: This will put a set of
	default security policies in play for your Fedora server.
	
	Options : true, false
	
	Enter a value [default is true] ==>
	
	
	Low Level Storage
	-----------------
	Which low-level (file) storage plugin do you want to use?
	We recommend akubra-fs for new installs.  If you are upgrading Fedora from
	version 3.3 or below, you should use legacy-fs for compatibility with your
	existing storage.  Other plugins are also available, but they must be
	configured after installation.
	
	Options : akubra-fs, legacy-fs
	
	Enter a value [default is akubra-fs] ==>
	
	
	Enable Resource Index
	---------------------
	Enable the Resource Index?
	
	Options : true, false
	
	Enter a value [default is false] ==>
	
	
	Enable Messaging
	----------------
	Enable Messaging? Messaging sends notifications of API-M events via JMS.
	
	Options : true, false
	
	Enter a value [default is false] ==> true
	
	
	Messaging Provider URI
	----------------------
	Please enter the messaging provider URI. For more information about
	using ActiveMQ broker URIs, see
	http://activemq.apache.org/broker-uri.html
	
	
	Enter a value [default is vm:(broker:(tcp://localhost:61616))] ==> tcp://localhost:61616
	
	
	Deploy local services and demos
	-------------------------------
	Several sample back-end services are included with this distribution.
	These are required if you want to use the demonstration objects.
	If you'd like these to be automatically deployed, enter 'true'.
	Otherwise, the installer will put the files in your FEDORA_HOME/install
	directory in case you want to deploy them later.
	
	Options : true, false
	
	Enter a value [default is true] ==> false
	
	
	Preparing FEDORA_HOME...
	  Configuring fedora.fcfg
	  Installing beSecurity
	Will not overwrite existing /opt/apache-tomcat-6.0.37/conf/server.xml.
	Wrote example server.xml to:
	  /opt/fedora/install/server.xml
	Preparing fedora.war...
	Deploying fedora.war...
	Installation complete.
	
	----------------------------------------------------------------------
	Before starting Fedora, please ensure that any required environment
	variables are correctly defined
	  (e.g. FEDORA_HOME, JAVA_HOME, JAVA_OPTS, CATALINA_HOME).
	For more information, please consult the Installation & Configuration
	Guide in the online documentation.
	----------------------------------------------------------------------


### Set the Environment Variables

**On the Server**

The following configuration files are in the Rails project config.

    hcsvlab / activemq_conf / activemq.xml      ---->  $ACTIVEMQ_HOME/conf/

            / fedora_conf / fedora.fcfg         ---->  $FEDORA_HOME/server/config/

            / tomcat_conf / setenv.sh           ---->  $CATALINA_HOME/bin/

            / solr_conf / hcsvlab               ---->  $SOLR_HOME/
                        / hcsvlab-solr.xml      ---->  $CATALINA_HOME/conf/Catalina/localhost/solr.xml
  
They need to be copied to the specified locations. This is performed by Capistrano as part of the `full_deploy` task.

Edit `.bashrc` and add the following:

    export RAILS_ENV=production
    export ACTIVEMQ_HOME=/opt/activemq
    export CATALINA_HOME=/opt/tomcat
    export FEDORA_HOME=/opt/fedora
    export SOLR_HOME=/opt/solr

### Deployment

Deployment is done from another machine to the target server setup in the steps above. The deployment machine can be your local desktop or any other machine.

**Install RVM on Deployment Machine**

    $ \curl -#L https://get.rvm.io | bash -s stable --autolibs=3 --ruby=2.0.0-p0
    $ source /home/devel/.rvm/scripts/rvm

**Download the WebApp**

On your deployment machine (which is probably your local machine, but can be another machine) clone the web app:

    $ git clone git@github.com:IntersectAustralia/hcsvlab.git
    
**Create a Gemset and Download Gems**

    $ cd projects/hcsvlab
    $ rvm use 2.0.0-p0@hcsvlab --create
    $ gem install bundler
    $ bundle

**Edit Configuration to Point to Target Server**

Open `config/deploy/production.rb` in a text editor, and change the hostnames to your server.

**Deploy**

    $ bundle exec cap production deploy:setup
    $ bundle exec cap production deploy:full_redeploy
    $ bundle exec cap production deploy:create_solr_core
    $ bundle exec cap production deploy:start_services

If you ever need to redeploy, make sure you run the following command to stop the services (ActiveMQ, Tomcat, messaging pollers) first: 

    $ bundle exec cap production deploy:stop_services

### Ingesting Data

Ingesting is the task of adding data to the system, which involves several processes such as: verifying and storing metadata and files, and creating indicies for search. 

Corpora are prepared for ingest using [RoboChef](https://github.com/IntersectAustralia/hcsvlab_robochef), which takes hetreogeneous metadata and normalises it, sets the URLs for the documents, and describes them in RDF. Preprepared data from another source can be ingested, but the URLs will all point to the host that was configured when it was RoboCheffed.

The command to ingest a corpus is `rake fedora:ingest`, which must be run from the web application's directory, and the path to the directory where the corpus and data files are located. For example, if you wanted to ingest the ace corpus, and it was stored in `/data/processed/ausnc/ace` then the command would be:

    
    $ rake fedora:ingest /data/processed/ausnc/ace
