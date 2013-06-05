
Install Postgres

    sudo yum install postgresql postgresql-libs postgresql-server
    sudo service postgresql initdb
    sudo su - postgres
    vi data/pg_hba.conf
    
Edit the last lines of the Postgres config as follows

    # "local" is for Unix domain socket connections only
    local all  all              trust
    # IPv4 local connections:
    host  all  all 127.0.0.1/32 password
    # IPv6 local connections:
    host  all  all ::1/128      password

Log out of the postgres user and run postgres as a service

    logout
    sudo chkconfig --level 345 postgresql on
    sudo service postgresql start
    
Create a Postgres user (adjust username and password as you wish)

    sudo su - postgres
    createuser -U postgres -P galaxy
    
Choose (and note down) a password. Answer y when prompted about whether the user should be a superuser.

Create a database (adjust username and database name as appropriate)

    createdb -U galaxy galaxy
