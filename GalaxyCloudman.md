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
9. Image: Alveo 1.00 (ami-00003389)
10. Cloudman will ask which type of cluster you want to start. Select ‘Galaxy Cluster’ and ‘Transient Storage’
11. Press 'Start an Instance'
12. Wait for launcher to finish booting up the machine and take note of the IP address it creates

####Configure Galaxy Instance

You can not SSH into your machine using the `ubuntu` user and the password you specified on the launch page.

**Updating Galaxy**

When you SSH into the machine, and old version of Galaxy will be mounted on `/msn/galaxy/galaxy-app`. To update the Galaxy version we first need to create a Galaxy user:

```
$ sudo adduser galaxy sudo
# if you are not prompted for a password:
$ sudo passwd galaxy
```

Now switch to the galaxy user:

```
$ su galaxy
```

Go to the mounted drive and clone the new Galaxy:

```
$ cd /mnt/galaxy
$ git clone https://github.com/galaxyproject/galaxy.git
```

Then checkout the desired release by tag, e.g:

```
$ cd galaxy
$ git checkout tags/v15.05
```

For convenience, softlink the galaxy directory to the galaxy user's home directory:

```
$ cd ~
$ ln -s /mnt/galaxy/galaxy galaxy
```

**Galaxy Configuration**

TODO

**Running Galaxy as a Service**

Copy the galaxy startup script to the service script directory:

```
$ sudo cp ~/galaxy/contrib/galaxy.debian-init /etc/init.d/galaxy
$ sudo chmod +x /etc/init.d/galaxy
```

Then edit the script to makes sure that paths point to galaxy. Copy the script and call it `galaxy-toolshed`, once again editing it to make sure the paths now point to the galaxy toolshed. Now start the services:

```
$ sudo service galaxy start
$ sudo service galaxy-toolshed start
```

Install Run-level configuration for SysV like init scripts

```
$ sudo apt-get install sysv-rc-config
```

Enable Galaxy and the Toolshed to run on system startup:

```
$ sudo sysv-rc-config galaxy on
$ sudo sysv-rc-config galaxy-toolshed on
```

### Manually Installed Tool Dependencies

**Set up libcurl so that pycurl can be built in the Alveo Importer tool**

    sudo apt-get install libcurl4-openssl-dev

**Set up XFVB, TkInter and ImageMagick for displaying NLTK's parse trees**

    sudo apt-get install python-tk -y
    sudo apt-get install imagemagick
    sudo apt-get install xvfb -y
    export DISPLAY=:1
    Xvfb :1 -screen 0 1024x768x24 &
    sudo xhost +
    sudo echo "export DISPLAY=:1" >> ~/.bashrc

**Install LibX and the Matlab Runtime for running PsySound Tools**

    sudo apt-get install libxt-dev
    sudo apt-get install libxmu-dev

Grab a copy of MCRInstaller.zip from the shared drive (or any existing server with the MCR installed). This is a large file (~400MB). Copy MCRInstaller.zip to the target server under the directory /mnt/galaxy/MATLAB. The file can be transferred using SCP or a similar mechanism. This will install MATLAB into /usr/local/MATLAB.

    cd /mnt/galaxy/MATLAB
    unzip MCRInstaller.zip
    sudo ./install -mode silent

**Set up server environment variable configuration**

    sudo vi /etc/ssh/sshd_config

Add the following line to `/etc/ssh/sshd_config`

    PermitUserEnvironment yes

Then run the following:

    sudo service ssh restart
    mkdir ~/.ssh
    sudo vi ~/.ssh/environment

Add the following line to `~/.ssh/environment`

    GALAXY_HOME=/mnt/galaxy/galaxy-app

**Set up Toolshed**

    sudo apt-get install sysv-rc-conf -y
    sudo createuser -U postgres -P toolshed
    sudo createdb -U postgres toolshed

    cd /mnt/galaxy/galaxy-app
    sudo cp contrib/galaxy.fedora-init /etc/init.d/toolshed
    sudo chmod 0755 /etc/init.d/toolshed
    sudo sysv-rc-conf toolshed on
    sudo vi /etc/init.d/toolshed

Modify the following lines in `/etc/init.d/toolshed`

    SERVICE_NAME="toolshed"
    RUN_AS="galaxy"
    RUN_IN="/mnt/galaxy/galaxy-app"

Replace all `run.sh` with `run_tool_shed`

    sudo sed -i 's/run\.sh/run_tool_shed\.sh/g' /etc/init.d/toolshed

Configure apache

    sudo update-rc.d toolshed defaults
    sudo a2enmod rewrite
    sudo sysv-rc-conf apache2 on

Add the following lines to `/etc/apache2/conf.d/toolshed.conf`

    RewriteEngine on
    RewriteRule ^/static/style/(.*) /home/toolshed/galaxy-dist/static/june_2007_style/blue/$1 [L]
    RewriteRule ^/static/scripts/(.*) /home/toolshed/galaxy-dist/static/scripts/packed/$1 [L]
    RewriteRule ^/static/(.*) /home/toolshed/galaxy-dist/static/$1 [L]
    RewriteRule ^/favicon.ico /home/toolshed/galaxy-dist/static/favicon.ico [L]
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

Turn off port listening on port 80
    sudo vi /etc/apache2/ports.conf
Remove the two lines NameVirtualHost:80 and Listen 80.

Restart apache

    sudo apachectl restart

**Set up Proxies for Galaxy and teh Toolshed**

SSH into the server running the Galaxy proxy, and edit the following:

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


### Smoke Test
You can verify that your install has been successful by executing these tests:

<table><tbody>
<tr>
<th> Test description </th>
<th> Pre-Conditions </th>
<th> Test steps </th>
<th> Expected Results </th>
</tr>
<tr>
<td> Galaxy requires login to see or use any tools/features </td>
<td>&nbsp;</td>
<td> 1. Open Galaxy <br/> </td>
<td> Galaxy should prompt user for a login </td>
</tr>
<tr>
<td> Login and see homepage </td>
<td>&nbsp;</td>
<td> 1. Open Galaxy <br/>
2. Login to Galaxy </td>
<td> User should see the Galaxy homepage with HCSVLAB logo and heading. The top left hand corner should also show "Galaxy / HCS vLab" </td>
</tr>
<tr>
<td> Upload file </td>
<td> Logged in </td>
<td> 1. Select Upload File from the tool menu (Get Data) <br/>
2. Choose a local text file <br/>
3. Click Execute </td>
<td> A new dataset should appear in the history on the RHS containing the contents of the uploaded file </td>
</tr>
<tr>
<td> Concatenator tool </td>
<td> Logged in </td>
<td> 1. Select Concatenator from the tool menu (Get Data) <br/>
2. Enter in a number of URL's pertaining to text files (one per line) <br/>
3. Click Execute </td>
<td> A new dataset should appear in the history on the RHS containing the text content of each URL combined into one output </td>
</tr>
<tr>
<td> Frequency List tool </td>
<td> Logged in, dataset available </td>
<td> 1. Select Frequency List from the tool menu (Analyse Data) <br/>
2. Select a dataset <br/>
3. Click Execute </td>
<td> A new dataset should appear in the history on the RHS containing a list of the words from the input dataset and how many times each occurred in that dataset </td>
</tr>
<tr>
<td> Sentence Segmenter tool </td>
<td> Logged in, dataset available </td>
<td> 1. Select Sentence Segmenter from the tool menu (NLTK Tools) <br/>
2. Select a dataset <br/>
3. Click Execute </td>
<td> A new dataset should appear in the history on the RHS containing each sentence of the input dataset split out onto one line </td>
</tr>
<tr>
<td> Tokenizer tool </td>
<td> Logged in, dataset available </td>
<td> 1. Select Tokenizer from the tool menu (NLTK Tools) <br/>
2. Select a dataset <br/>
3. Click Execute </td>
<td> A new dataset should appear in the history on the RHS containing each token of the input dataset split out onto one line </td>
</tr>
<tr>
<td> POS tool </td>
<td> Logged in, dataset available </td>
<td> 1. Select POS from the tool menu (NLTK Tools) <br/>
2. Select a dataset <br/>
3. Click Execute </td>
<td> A new dataset should appear in the history on the RHS containing each token of the input dataset split out onto one line paired with its POS tag e.g. this/DT </td>
</tr>
<tr>
<td> Stemmer tool </td>
<td> Logged in, dataset available </td>
<td> 1. Select Stemmer from the tool menu (NLTK Tools) <br/>
2. Select a dataset and a stemmer option <br/>
3. Click Execute </td>
<td> A new dataset should appear in the history on the RHS containing a list of stems (words/tokens) each split out onto one line </td>
</tr>
<tr>
<td> Collocation tool </td>
<td> Logged in, dataset available </td>
<td> 1. Select Collocation from the tool menu (NLTK Tools) <br/>
2. Select a dataset and a set of options <br/>
3. Click Execute </td>
<td> A new dataset should appear in the history on the RHS containing a set of the most frequent collocations (a pair/set of three words) each split out onto one line e.g. ('this', 'is') </td>
</tr>
<tr>
<td> Chart Parser tool </td>
<td> Logged in, dataset available, valid grammar available </td>
<td> 1. Select Chart Parser from the tool menu ( NLTK Tools) <br/>
2. Select a grammar and a dataset <br/>
3. Click Execute </td>
<td> A new dataset should appear in the history on the RHS containing a text output of the tree of the parsed input </td>
</tr>
<tr>
<td> Johnson-Charniak tool </td>
<td> Logged in, dataset available </td>
<td> 1. Select Johnson-Charniak Parser from the tool menu (Johnson-Charniak Parser Tools) <br/>
2. Select a dataset <br/>
3. Click Execute </td>
<td> A new dataset should appear in the history on the RHS containing a textual and graphical output of the parsed input dataset </td>
</tr>
</tbody></table>

