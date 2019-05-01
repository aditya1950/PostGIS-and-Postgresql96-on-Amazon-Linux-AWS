# Installing PostGIS and Postgresql96 on Amazon Linux 2018.03 AWS
Here we are going to install PostGIS and Postgresql96 on Amazon Linux. 
PostgreSQL is a powerful, open source object-relational database system.
PostGIS is a spatial database extender for PostgreSQL object-relational database. It adds support for geographic objects allowing location queries to be run in SQL.

## Getting Started
These instructions will get you install PostGIS and Postgresql96 on your Amazon Linux Machine for development, testing and production purposes.

### Prerequisites

```
1. Up and running EC2 instance
2. Login access to the EC2 instance
```

### Installation

Login to the console and install postgresql, postgresql-devel, and postgresql-libs, and configure data directory and start the service (as the postgres user).

### PostgreSQL96 installation

```
sudo yum install postgresql96 postgresql96-server postgresql96-devel postgresql96-contrib postgresql96-docs postgresql96-libs

Initialize database:
sudo service postgresql96 initdb

Edit your pg_hba.conf file:
sudo vim /var/lib/pgsql96/data/pg_hba.conf

Scroll down until you see something like this, by default:

# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     ident
# IPv4 local connections:
host    all             all             127.0.0.1/32            ident
# IPv6 local connections:
host    all             all             ::1/128                 ident

Change it to read like this:
# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     trust
# IPv4 local connections:
host    all             username	        0.0.0.0/0               md5
host    all             other_username    0.0.0.0/0               md5
# IPv6 local connections:
host    all             all             ::1/128                 md5

Enter username for each user and associate IPs or make it open to all 0.0.0.0/0
Enter a specific IP X.X.X.X/32 to only allow connection from the exact IP

Now, we need to update PostgreSQL to enable remote connections to the database. At the command line enter:

sudo vim /var/lib/pgsql96/data/postgresql.conf

Uncomment line:
#listen_addresses = 'localhost'          # what IP address(es) to listen on;

Update that line to enable connections from any IP addresses:
listen_addresses='*'

Uncomment line:
#port = 5432

To read like this:
port = 5432

Start the postgresql service:
sudo service postgresql start

Log into the postgresql

sudo su - postgres
psql -U postgres

Add a password for your PostgreSQL admin

ALTER USER postgres WITH PASSWORD '$password';

Create user credentials for different users.
Replace username with the username you want. Replace $password with the password you want.
CREATE USER username SUPERUSER;
ALTER USER username WITH PASSWORD '$password';

CREATE USER username NOSUPERUSER;
ALTER USER username WITH PASSWORD '$otheruserpassword';

Exit from Postgres with \q. Your setup is complete now and is ready to be connected remotely.
```
