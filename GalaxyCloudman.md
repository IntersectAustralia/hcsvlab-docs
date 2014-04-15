### Background

These are instructions to setup a new Nectar VM with Galaxy and Cloudman installed

### Setup

**Access the Australian Research Cloud**

1. Login via the [Dashboard](https://dashboard.rc.nectar.org.au)
2. Click on 'Access & Security’ link in the left hand panel and then the ‘API Access’ tab
3. Click on ‘Download EC2 Credentials’ at top right to download a zip archive of the credentials for that Project
4. Unzip the downloaded archive. Open the file 'ec2rc.sh' with a text editor. You will need `EC2_ACCESS_KEY` and `EC2_SECRET_KEY` each time you use the Launch page (next Section).

**Launch the CloudBioLinux/Cloudman image**

1. Go to the [launch page](http://launch.genome.edu.au), bringing you to the GVL BioCloudCentral launch page.
2. Cloud: Select ‘NeCTAR (Openstack)’
3. Access key and Secret key: Obtained in the previous Section.
4. Institional Email: Something like "enquiries@intersect.org.au"
5. Cluster Name: Choose a name for your GVL instance. Whatever you want. Avoid using a cluster name you have previously used
6. Password: Choose a password.  Whatever you want, just remember it. This is the password you will use to log into the Cloudman instance
7. Instance Type: Choose Medium (2 vcpu / 8GB RAM) CPU/RAM size.
8. Expand 'Show Advanced Startup Options
9. Image: GVL-2.09 (November 16, 2013) (ami-00001303)
10. Press 'Start an Instance'
11. Wait for launcher to finish booting up the machine and take note of the IP address it creates

**Start a Galaxy instance**

1. Using the IP address obtained previously go to <IP address>/cloud
2. Log in to Cloudman using username ‘ubuntu’ and the password you chose on the launch page. You are now taken to the Cloudman Admin Console page
3. Cloudman will ask which type of cluster you want to start. Select ‘Galaxy Cluster’ and ‘Transient Storage’
4. Press “Choose Platform Type’

####Configure Galaxy Instance

**Install the tool wrappers and customisations from Github**

ssh into the machine with the user ubuntu and perform the following:

    cd /mnt/galaxy/galaxy-app
    sudo git init
    sudo git remote add origin git://github.com/IntersectAustralia/hcsvlab-galaxy.git
    sudo git fetch --all
    sudo git reset --hard origin/master
    sudo cp shed_tool_conf.xml.sample shed_tool_conf.xml
    
    sudo adduser galaxy sudo
    
Log out of the machine and ssh back in as the galaxy user> Run the following:

    cd /mnt/galaxy
    sudo chown -R galaxy galaxy-app
    
**Set up R for ParseEval**

    sudo R
    install.packages("lattice", repos="http://cran.ms.unimelb.edu.au/")
    install.packages("latticeExtra", repos="http://cran.ms.unimelb.edu.au/")
    install.packages("gridExtra", repos="http://cran.ms.unimelb.edu.au/")
    
**Install NLTK**

    sudo pip install -U numpy
    sudo easy_install -U distribute
    sudo pip install -U pyyaml nltk
    sudo mkdir /usr/share/nltk_data
    sudo python -m nltk.downloader -d /usr/share/nltk_data all

**Install the Johnson Charniak Parser**

    sudo apt-get update
    sudo apt-get install flex
    sudo cd /mnt/galaxy
    sudo wget http://web.science.mq.edu.au/~mjohnson/code/reranking-parser-2011-12-17.tgz
    sudo tar -zxvf reranking-parser-2011-12-17.tgz
    sudo chown -R galaxy reranking-parser
    
There is a line missing in `reranking-parser/second-stage/programs/features/best-parses.cc` - edit the file and add the following near other #includes:

    #include <unistd.h>

Compile

    cd reranking-parser
    sudo make

**Install the JC Parser NLTK wrapper**

    sudo apt-get install python-tk
    
Clone the repository

    sudo git clone git://github.com/IntersectAustralia/jcp-nltk-wrapper.git
    
Now you need to find where NLTK is installed. This is likely to be something like `/usr/local/lib/python2.7/dist-packages/nltk/parse` - adjust paths below if yours is different.    

Copy the files under `/mnt/galaxy/jcp-nltk-wrapper/parse` to the python NLTK installation directory as per above

    sudo cp /mnt/galaxy/jcp-nltk-wrapper/parse/* /usr/local/lib/python2.7/dist-packages/nltk/parse
    
Edit `/usr/local/lib/python2.7/dist-packages/nltk/parse/johnsoncharniak.ini` to point your installation of the Johnson Charniak parser, which should be ~/reranking-parser

    basedir: /mnt/galaxy/reranking-parser
    
Edit `/usr/local/lib/python2.7/dist-packages/nltk/parse/johnsoncharniak.py` so that the `config.read` line points your edited copy of johnsoncharniak.ini

    config.read('/usr/local/lib/python2.7/dist-packages/nltk/parse/johnsoncharniak.ini')

**Set up for displaying parse trees**

    sudo apt-get install xvfb
    export DISPLAY=:1
    Xvfb :1 -screen 0 1024x768x24 &
    sudo xhost +
    sudo echo "export DISPLAY=:1" >> ~/.bashrc
    
**Install PsySound**

Grab a copy of MCRInstaller.zip from the shared drive (or any existing server with the MCR installed). This is a large file (~400MB). Copy MCRInstaller.zip to the target server under the directory ~/MATLAB. The file can be transferred using SCP or a similar mechanism.

    cd ~/MATLAB
    unzip MCRInstaller.zip
    sudo ./install -mode silent

**Set up server environment variable configuration**

    sudo vi /etc/ssh/sshd_config
    
Add the following line to `/etc/ssh/sshd_config`

    PermitUserEnvironment yes

Then run the following:

    sudo service ssh restart
    mkdir ~/.ssh
    sudo vi .ssh/environment
    
Add the following line to `.ssh/environment`

    GALAXY_HOME=/mnt/galaxy/galaxy-app

**Set up Toolshed**

    sudo createuser -U postgres -P toolshed
    sudo createdb -U postgres toolshed
    
    cd /mnt/galaxy/galaxy-app
    sudo cp contrib/galaxy.fedora-init /etc/init.d/toolshed
    sudo chmod 0755 /etc/init.d/toolshed
    sudo chkconfig --add toolshed
    sudo vi /etc/init.d/toolshed
    
Modify the following lines in `/etc/init.d/toolshed`

    SERVICE_NAME="toolshed"
    RUN_AS="galaxy"
    RUN_IN="/mnt/galaxy/galaxy-app"

Replace all `run.sh` with `run_tool_shed`

    sudo sed -i 's/run\.sh/run_tool_shed\.sh/g' /etc/init.d/toolshed

Configure apache

    sudo update-rc.d toolshed defaults
    sudo apt-get install sysv-rc-conf
    sudo sysv-rc-conf apache2 on
    
Add the following lines to `/etc/apache2/conf.d/toolshed.conf`

    RewriteEngine on
    RewriteRule ^/static/style/(.*) /home/toolshed/galaxy- dist/static/june_2007_style/blue/$1 [L]
    RewriteRule ^/static/scripts/(.*) /home/toolshed/galaxy- dist/static/scripts/packed/$1 [L]
    RewriteRule ^/static/(.*) /home/toolshed/galaxy-dist/static/$1 [L] 
    RewriteRule ^/favicon.ico     /home/toolshed/galaxy-dist/static/favicon.ico [L] 
    RewriteRule ^/robots.txt /home/toolshed/galaxy-dist/static/robots.txt [L] 
    RewriteRule ^(.*) http://localhost:9009$1 [P] 
    
Configure tool shed server

    sudo cp tool_sheds_conf.xml.sample tool_sheds_conf.xml
    sudo cp shed_tool_conf.xml.sample shed_tool_conf.xml
    sudo vi tool_sheds_conf.xml
    
Add the following inside the `toolsheds` tag

    <tool_shed name="HCS vLab tool shed" url="__TOOL_SHED_URL__"/>

Start it up

    sudo service toolshed start

**Set up proxies for galaxy and toolshed**

Ssh into the server where the webapp is

    sudo nano /etc/httpd/conf.d/galaxy_vhost.conf
    
Add the following lines to `/etc/httpd/conf.d/galaxy_vhost.conf`

    ## Galaxy Proxy
    Listen 8081
    NameVirtualHost *:8081
    <VirtualHost *:8081>
         ServerName <SERVER_NAME>
    
         <Proxy *>
                Order deny,allow
                Allow from all
         </Proxy>
         ProxyPass / http://<NECTAR_VM_IP>:8080/
         ProxyPassReverse / http://<NECTAR_VM_IP>:8080/
    </VirtualHost>
    
    ## Toolshed Proxy
    Listen 9009
    NameVirtualHost *:9009
    <VirtualHost *:9009>
         ServerName <SERVER_NAME>
    
         <Proxy *>
                Order deny,allow
                Allow from all
         </Proxy>
         ProxyPass / http://<NECTAR_VM_IP>:9009/
         ProxyPassReverse / http://<NECTAR_VM_IP>:9009/
    </VirtualHost>

where you replace `<SERVER_NAME>` with the server you're on (where the webapp is) and `<NECTAR_VM_IP>` with the IP address of the nectar VM with Galaxy w/ cloudman installed

Restart apache

    sudo apachectl restart

