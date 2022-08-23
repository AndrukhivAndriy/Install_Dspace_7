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

## Install Apache Solr

We are looking for v 8.11 or above. NOT v9
Create a directory to download and install solr: *sudo mkdir /opt/solr*

Then download solr: *wget -c https://downloads.apache.org/lucene/solr/8.11.2/solr-8.11.2.tgz*

Untar it and start: *../bin/solr start &* . Confirm solr is running with: *../bin/solr status*

It can't start under root user (parametr "-force" it's not good for security), so then we will change it

 ##Install Apache Tomcat v9
 
 Use the following command to install Apache Tomcat: *sudo apt install tomcat9*
 
 Edit /etc/default/tomcat9 and define JAVA_HOME and JAVA_OPTS: 
 
 1. *JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64*
 2. *JAVA_OPTS="-Djava.awt.headless=true -Xmx6G -Xms2G -Dfile.encoding=UTF-8"*,
 
 where -Xmx -- specifies the maximum memory allocation pool for a JVM, Xms -- specifies the initial memory allocation pool
 
 Locate the setting JAVA_OPTS settings accordingly depending on how much RAM your system has.

Edit the file */etc/tomcat9/server.xml* by editing as shown. Ensure that the active connector element in the file is similar to the one below:

        <Connector port="8080"
              minSpareThreads="25"
              enableLookups="false"
              redirectPort="8443"
              connectionTimeout="20000"
              disableUploadTimeout="true"
              URIEncoding="UTF-8"/>
              
Restart tomcat9 as shown below: *sudo systemctl restart tomcat9*
When you access the ip address of your system via the browser at port 8080 you should see the default tomcat page, **It works !**. 

If not:

1. Execute the following command to check ports for apache tomcat server: *ss -ltn*
2. Execute the following command to permit the incoming from any type of source to port “8080”: *sudo ufw allow from any to any port 8080 proto tcp*
3. Check again( also you can check via text browser LYNX - *apt install lynx* and then type: *lynx 127.0.0.1:8080*)

## Install DSPACE Backend

Get the latest DSpace 7.3 release and extract it:

1. *mkdir /opt/dspaceinst*
2. *cd /opt/dspaceinst* 
3. *wget -c https://github.com/DSpace/DSpace/archive/refs/tags/dspace-7.3.tar.gz*
4. *tar zxvf dspace-7.3.tar.gz*

Create a configuration file by copying the existing example file: *cp /opt/dspaceinst/DSpace-dspace-7.3/dspace/config/local.cfg.EXAMPLE /opt/dspaceinst/DSpace-dspace-7.3/dspace/config/local.cfg*

Edit the file */opt/dspaceinst/DSpace-dspace-7.3/dspace/config/local.cfg* and set some common required settings:

     dspace.dir=/home/dspace-7
     solr.server = http://localhost:8983/solr
     db.url = jdbc:postgresql://localhost:5432/dspace
     db.driver = org.postgresql.Driver
     db.username = dspace
     db.password = secure_password_here
     
## Build and install
     
