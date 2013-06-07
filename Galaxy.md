### Background
These instructions are to install Galaxy and the HCSvLab tools on a CentOS machine. This can easily be adapted to other distributions.

The Galaxy instructions are based on the following from the Galaxy website
* [Basic install](http://wiki.galaxyproject.org/Admin/Get%20Galaxy) 
* [Running in production](http://wiki.galaxyproject.org/Admin/Config/Performance/ProductionServer)

### Assumptions
* You have a fresh CentOS machine. 
* You have a non-root user account on this machine with sudo privileges. We used "galaxy".
* You are logged in as this user and are in the home directory.

### Setup
**Install development tools and mercurial**

    sudo yum groupinstall "Development Tools"
    sudo yum install mercurial

**Pull down Galaxy from mercurial and select the stable branch**

    hg clone https://bitbucket.org/galaxy/galaxy-dist/
    cd galaxy-dist
    hg update stable

**Install Tools**

[Install NLTK](NLTK.md) (you should install this as the same user that will run Galaxy)

[Install the Johnson Charniak Parser](JCParser.md) (you should install this as the same user that will run Galaxy)

[Install the JC Parser NLTK wrapper](JCPNLTKWrapper.md) (you should install this as the same user that will run Galaxy)

**Set up for displaying parse trees**

    sudo yum install tkinter python-matplotlib-tk.x86_64 xorg-x11-server-Xvfb ImageMagick
    Xvfb :1 -screen 0 1024x768x24 &
    export DISPLAY=:1
    sudo yum install xhost
    xhost +
    
Add the `DISPLAY` configuration to your bashrc

    cd ~
    vi .bashrc
    
Paste the following line into the file:

    export DISPLAY=:1
    
Then reload your bashrc    

    source ~/.bashrc

**Install HCSvLab Galaxy customisations and tool wrappers**

Install the tool wrappers and customisations from Github

    cd ~/galaxy-dist
    git init
    git remote add origin git://github.com/IntersectAustralia/hcsvlab-galaxy.git
    git pull origin master
    chmod 755 galaxy
    
Modify the start script with user and path - modify the `RUN_AS` and `RUN_IN` to suit your path and user    

    vi galaxy
    
Start Galaxy    

    ./galaxy start
    
### Production Configuration    
**If you are just trying it out or running it locally for development, you can stop here.** If running in production, we recommend the following steps (which come from the Galaxy recommendations for running in production). We suggest you review the Galaxy documentation and choose options that are most appropriate to your local installation. Below is what we did.

**Turn off development settings**

    vi ~/galaxy-dist/universe_wsgi.ini
   
and modify the following config items to False:

    debug = False
    use_interactive = False

**Install and configure Postgres**

Install Postgres, create a Postgres user called "galaxy" and create a database called "galaxy". If you need guidance on this see [Install Postgres](Postgres.md)

Configure Galaxy to use Postgres

    vi ~/galaxy-dist/universe_wsgi.ini
    
uncomment the `database_connection` configuration. Modify it for your postgres settings. This should be something like:

    database_connection = postgresql://<user>:<password>@localhost:5432/<dbname>
    
update the database schema

    cd ~/galaxy-dist
    sh manage_db.sh upgrade
    
**Install and configure Apache**

    sudo yum install httpd httpd-devel
    sudo vi /etc/httpd/conf.d/galaxy.conf

and add the following (adjusting paths as needed):

    RewriteEngine on
    RewriteRule ^/static/style/(.*) /home/galaxy/galaxy-dist/static/june_2007_style/blue/$1 [L]
    RewriteRule ^/static/scripts/(.*) /home/galaxy/galaxy-dist/static/scripts/packed/$1 [L]
    RewriteRule ^/static/(.*) /home/galaxy/galaxy-dist/static/$1 [L]
    RewriteRule ^/favicon.ico /home/galaxy/galaxy-dist/static/favicon.ico [L]
    RewriteRule ^/robots.txt /home/galaxy/galaxy-dist/static/robots.txt [L]
    RewriteRule ^(.*) http://localhost:8080$1 [P]

**Restart Galaxy & Apache**

Make sure that the apache user has read/execute permissions to the galaxy directory ~/galaxy-dist

    chmod go+rx ~

Set up Galaxy as a service, then restart Galaxy and Apache.

    sudo yum install python-psycopg2
    sudo cp ~/galaxy-dist/galaxy /etc/init.d/
    sudo chkconfig --level 345 galaxy on
    sudo chkconfig --level 345 httpd on
    sudo service httpd restart
    sudo service galaxy restart

TODO: turn off SELinux


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
    
