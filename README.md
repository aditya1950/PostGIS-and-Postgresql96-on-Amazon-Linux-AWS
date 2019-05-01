# Installing PostGIS and Postgresql96 on Amazon Linux 2018.03 AWS
Here we are going to install PostGIS and Postgresql96 on Amazon Linux. 
PostgreSQL is a powerful, open source object-relational database system.
PostGIS is a spatial database extender for PostgreSQL object-relational database. It adds support for geographic objects allowing location queries to be run in SQL.

## Getting Started
These instructions will get you install PostGIS and Postgresql96 on your Amazon Linux Machine for development, testing and production purposes.

### Prerequisites

```
1. Up and running EC2 instance
2. Login access to the EC2 instance using Putty
```

### Installation

Login to the console and install postgresql, postgresql-devel, and postgresql-libs, and configure data directory and start the service (as the postgres user).

### PostgreSQL96 installation

```
--Update your system
sudo yum update

--Install PostgreSQL96
sudo yum install postgresql96 postgresql96-server postgresql96-devel postgresql96-contrib postgresql96-docs postgresql96-libs

--Initialize database:
sudo service postgresql96 initdb

--Edit your pg_hba.conf file:
sudo vim /var/lib/pgsql96/data/pg_hba.conf

--Scroll down until you see something like this, by default:

# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     ident
# IPv4 local connections:
host    all             all             127.0.0.1/32            ident
# IPv6 local connections:
host    all             all             ::1/128                 ident

--Change it to read like this:
# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     trust
# IPv4 local connections:
host    all             username	        0.0.0.0/0               md5
host    all             other_username    0.0.0.0/0               md5
# IPv6 local connections:
host    all             all             ::1/128                 md5

--Enter username for each user and associate IPs or make it open to all 0.0.0.0/0
--Enter a specific IP X.X.X.X/32 to only allow connection from the exact IP

--Now, we need to update PostgreSQL to enable remote connections to the database.

sudo vim /var/lib/pgsql96/data/postgresql.conf

--Uncomment line:
#listen_addresses = 'localhost'          # what IP address(es) to listen on;

--Update that line to enable connections from any IP addresses:
listen_addresses='*'

--Uncomment line:
#port = 5432

--To read like this:
port = 5432

--Start the postgresql service:
sudo service postgresql start

--Log into the postgresql

sudo su - postgres
psql -U postgres

--Add a password for your PostgreSQL admin

ALTER USER postgres WITH PASSWORD '$password';

--Create user credentials for different users.
--Replace username with the username you want. Replace $password with the password you want.
CREATE USER username SUPERUSER;
ALTER USER username WITH PASSWORD '$password';

CREATE USER username NOSUPERUSER;
ALTER USER username WITH PASSWORD '$otheruserpassword';

--Exit from Postgres with \q. Your setup is complete now and is ready to be connected remotely.
--Postgresql96 install done
```
### PostGIS installation

First, we install some build tools and the GEOS and PROJ libraries. Then we install PostGIS. Once installed, we will update our libraries, so the server knows where to find them. Finally, we create a temp database for PostGIS.

```
--Install build tools
sudo yum install gcc make gcc-c++ libtool libxml2-devel

--Make a directory for PostGIS
cd /home/ec2-user/
mkdir postgis
cd postgis

--Install GEOS - 3.7.1
wget http://download.osgeo.org/geos/geos-3.7.1.tar.bz2
tar xjvf geos-3.7.1.tar.bz2
cd geos-3.7.1
./configure
make
sudo make install

--Install PROJ - 6.0.0 and datumgrid-1.7 
--Find latest build here  - https://proj4.org/install.html#install
--This build requires sqlite3 headers - if the PROJ configure does not work (see below), install sqlite3 from the website - get the latest version
--Follow this installation - https://www.tutorialspoint.com/sqlite/sqlite_installation.htm

--SQLite3 install
--cd into the folder you want to download
wget https://www.sqlite.org/2019/sqlite-autoconf-3280000.tar.gz
tar xvfz sqlite-autoconf-3280000.tar.gz
cd sqlite-autoconf-3280000
./configure --prefix=/usr/local
make
make install

--set the PKG_CONFIG_PATH to find the sqlite3.pc file

export PKG_CONFIG_PATH=/your/path/name

--In my case - export PKG_CONFIG_PATH=/usr/local/lib/pkgconfig

--Install PROJ - 6.0.0 and datumgrid-1.7
cd /home/ec2-user/postgis/
wget http://download.osgeo.org/proj/proj-6.0.0.tar.gz
wget http://download.osgeo.org/proj/proj-datumgrid-1.7.zip
tar zxvf proj-6.0.0.tar.gz
cd proj-6.0.0/data
unzip ../../proj-datumgrid-1.7.zip
cd ..
./configure  
--Follow the steps to install SQLite3 above 
make
sudo make install

--Install GDAL - 2.4.1
cd /home/ec2-user/postgis
wget http://download.osgeo.org/gdal/gdal-2.4.1.tar.gz
gzip -d gdal-2.4.1.tar.gz
tar -xvf gdal-2.4.1.tar
cd gdal-2.4.1
./configure
make
sudo make install

--Install postgis-2.5.2
--Download the latest tar file from https://postgis.net/source/
--postgis-2.5.2.tar

cd /home/ec2-user/postgis/
wget http://download.osgeo.org/postgis/source/postgis-2.5.2.tar.gz
tar zxvf postgis-2.5.2.tar.gz 
cd postgis-2.5.2
./configure --with-geosconfig=/usr/local/bin/geos-config
make
sudo make install

--Create a temporary database to test

createdb -U postgres temp_postgis
createlang -U postgres plpgsql temp_postgis
psql -U postgres -d temp_postgis < /usr/share/pgsql96/contrib/postgis-2.5/postgis.sql
psql -U postgres -d temp_postgis < /usr/share/pgsql96/contrib/postgis-2.5/spatial_ref_sys.sql
```
Once everything is installed, open your choice of Postgresql software and login or login via CLI (Command Line Interface).

### Enabling PostGIS
PostGIS is an optional extension that must be enabled in each database you want to use it in before you can use it. Installing the software is just the first step. DO NOT INSTALL it in the database called postgres.

Connect to your database with psql or PgAdmin. Run the following SQL. You need only install the features you want:


```
-- Enable PostGIS (includes raster)
CREATE EXTENSION postgis;
-- Enable Topology
CREATE EXTENSION postgis_topology;
-- Enable PostGIS Advanced 3D
-- and other geoprocessing algorithms
-- sfcgal not available with all distributions
CREATE EXTENSION postgis_sfcgal;
-- fuzzy matching needed for Tiger
CREATE EXTENSION fuzzystrmatch;
-- rule based standardizer
CREATE EXTENSION address_standardizer;
-- example rule data set
CREATE EXTENSION address_standardizer_data_us;
-- Enable US Tiger Geocoder
CREATE EXTENSION postgis_tiger_geocoder;
```

### Sources
[PostgreSQL](https://www.postgresql.org/)
[PostGIS](https://postgis.net/install/)
(http://en.joysword.com/posts/2015/05/configuring_geo_spatial_stack_on_amazon_linux/)
