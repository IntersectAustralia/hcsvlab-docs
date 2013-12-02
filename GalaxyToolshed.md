### Background
These instructions are to install an instance of Galaxy Tool Shed alongside an existing setup of Galaxy

### Assumptions
* You have a fresh CentOS machine.
* Port 9009 is open
* Galaxy is already installed as per instructions found [here](https://github.com/IntersectAustralia/hcsvlab-docs/blob/master/Galaxy.md)

### Setup

**Create database user**

    createuser -U postgres -P toolshed
    
**Create database**

    createdb -U toolshed toolshed
    
**Install toolshed as a service**

    cd hcsvlab-galaxy
    sudo cp contrib/galaxy.fedora-init /etc/init.d/toolshed
    sudo chmod 0755 /etc/init.d/toolshed
    sudo chkconfig --add toolshed
    sudo vi /etc/init.d/toolshed
    
Modify the following lines in `/etc/init.d/toolshed`
  
    SERVICE_NAME="toolshed"
    RUN_AS="devel"
    RUN_IN="/home/devel/hcsvlab-galaxy"

Replace all `run.sh` with `run_tool_shed`

    sudo sed -i 's/run\.sh/run_tool_shed\.sh/g' /etc/init.d/toolshed
    
**Configure Apache**

    sudo chkconfig httpd on
    chmod a+rx /home/devel
    sudo vi /etc/httpd/conf.d/toolshed.conf 
    
Add the following lines to `/etc/httpd/conf.d/toolshed.conf`

    RewriteEngine on
    RewriteRule ^/static/style/(.*) /home/toolshed/galaxy- dist/static/june_2007_style/blue/$1 [L]
    RewriteRule ^/static/scripts/(.*) /home/toolshed/galaxy- dist/static/scripts/packed/$1 [L]
    RewriteRule ^/static/(.*) /home/toolshed/galaxy-dist/static/$1 [L] 
    RewriteRule ^/favicon.ico     /home/toolshed/galaxy-dist/static/favicon.ico [L] 
    RewriteRule ^/robots.txt /home/toolshed/galaxy-dist/static/robots.txt [L] 
    RewriteRule ^(.*) http://localhost:9009$1 [P]
    
**Configure tool shed server**

    cp tool_sheds_conf.xml.sample tool_sheds_conf.xml
    cp shed_tool_conf.xml.sample shed_tool_conf.xml
    vi tool_sheds_conf.xml
    
Add the following inside the `toolsheds` tag

    <tool_shed name="HCS vLab tool shed" url="__TOOL_SHED_URL__"/>
    
**Install Python psycopg2 module**

Seems to need this python module installed to get it up and going

    sudo yum install python-psycopg2
    
**Redeploy Galaxy and Galaxy Tool Shed**

Run the `deploy:redeploy_galaxy` and `deploy:redeploy_galaxy_toolshed` tasks from the deployment server
