###Prototype Developer Setup

    $ git clone git@github.com:IntersectAustralia/hcsvlab-prototype.git
    $ cd hcsvlab
    $ rvm use 1.9.3-p286@hcsvlab --create
    $ bundle install
    $ rake db:create
    $ rake db:migrate
    $ rake sunspot:solr:start
    $ rake db:seed
    $ rails s

***If execjs can't find a js runtime & you're on Ubuntu***

    $ sudo apt-get install nodejs