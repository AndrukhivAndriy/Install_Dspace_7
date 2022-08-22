# Install/Upgrade Dspace 7.3 on Ubuntu 22

The official manual you can find - https://wiki.lyrasis.org/display/DSDOC7x/Installing+DSpace

The first and main step is upgrade system: *apt update && apt upgrade*

## Install JAVA 11

We shall install OpenJDK 11 (with JAVA 17 there were some bugs. I don't recommend this version): *sudo apt install openjdk-11-jdk*

To confirm the version installed on the system, type: *java -version*

You will see thomething like this: *openjdk version "11.0.16" 2022-07-19 ....*

## Install Apache Maven and Ant

DSpace 7.3 requires maven 3.3.x or above and 1.10.x or above

To install type: *sudo apt install ant ant-optional maven*

To check the versions installed, see the following commands: *mvn -v*

You will see thomething like this: *Apache Maven 3.6.3 Maven home: /usr/share/maven ....*

And for Ant type: *ant -version*

You will see thomething like this: *Apache Ant(TM) version 1.10.12 compiled on ....*

## Install PostgreSQL Database v13

We will install v13 because the final release will be on November 13, 2025. For v11 - November 9, 2023

To install it, we shall first add its repository signing key: *curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc|sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/postgresql.gpg*

Then add the actual repository contents to Ubuntu 22: *echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" |sudo tee  /etc/apt/sources.list.d/pgdg.list*

Update the system to read the list of packages: *sudo apt update*

Then install PostgreSQL 13: *sudo apt install postgresql-13 postgresql-contrib-13 libpostgresql-jdbc-java*


### Config Postgresql


Add the following line to the bottom of the file */etc/postgresql/11/main/pg_hba.conf* then save and close: *host    dspace       dspace      127.0.0.1/32         md5*

We have to create the dspace database user and the dspace database: *sudo su postgres* 

Then create the dspace database user with the following command: *createuser dspace*

Now create the dspace database: *createdb dspace -E UNICODE*

We need to grant permissions to the database user called dspace: *psql -d dspace*, then tyoe *CREATE EXTENSION pgcrypto;*

Create the password: *ALTER ROLE dspace WITH PASSWORD 'secure_password_here';*

Then give the dspace database user ownership of the dspace database: *ALTER DATABASE dspace OWNER TO dspace;*

Then give all privileges to dspace database user on the dspace database: *GRANT ALL PRIVILEGES ON DATABASE dspace TO dspace;*

Exit the Postgresql: *\q*

Exit the postgres user session: *exit*

Restart Postgresql: *systemctl restart postgresql*
