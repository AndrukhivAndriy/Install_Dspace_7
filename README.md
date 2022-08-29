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
     
Ensure you are in the source code directory */opt/dspaceinst/DSpace-dspace-7.3* and run: *mvn package*

At the end you will see: *....BUILD SUCCESS .....*

Then we install Dspace. Type: *cd /opt/dspaceinst/DSpace-dspace-7.3/dspace/target/dspace-installer*

And run: *ant fresh_install*

At the end you will see: * .... BUILD SUCCESSFUL ....*

Copy */dspace/webapps/server* to */var/lib/tomcat9/webapps*

Change owner to tomcat user: * chown -R tomcat:tomcat /var/lib/tomcat9/webapps*

Copy Solr configsets */dspace/solr/ALL_DIRS* to *opt/solr/server/solr/configsets* 

Initialize database: */dspace/bin/dspace database migrate*

Start Solr under noroot user: *su USER -c '/opt/solr/bin/solr start &'*

Restart Tomcat go to http://server-ip:8080/server . You will see HALL BROWSER

Create an initial administrator account: *dspace/bin/dspace create-administrator*

## Migrate from v6.3 to 7.3

Dump database from server where dspace v6.3 installed: *pg_dump -h localhost -U dspace > /home/dspace.sql*

###Restore this database on new server:

1. *su postgres*
2. *psql* 
3. *REVOKE CONNECT ON DATABASE dspace FROM public;*
4. *SELECT pg_terminate_backend(pg_stat_activity.pid) FROM pg_stat_activity WHERE pg_stat_activity.datname = 'dspace';*
5. *\q*
6. *dropdb dspace*
7. *sudo service postgresql restart* 
8.  *createdb dspace -E UNICODE*
9.  *psql -d dspace*, then tyoe *CREATE EXTENSION pgcrypto;*
10.  *\q*
11.  Copy dumped file dspace.sql to new server to folder /home
12.  *psql -h localhost -U dspace < /home/dump.sql*
13.  */dspace/bin/dspace database migrate ignored*
14.  copy folder /dspace/assetstore from dspace v6.3 to the same place v7.3

## Install frontend

### Install Node.js (v14.x or v16.x). We will install v14.

Will install via node version manager. Go to this link *https://github.com/creationix/nvm/releases* to check the current stable release.

Run *curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.39.1/install.sh | bash*. This will install nvm v0.39.1

Reload your terminal session and type the command below just to ensure that nvm installed correctly: *nvm --version*

To see node.js version LTS, type: *nvm ls-remote*. Check version to install and type like this: *nvm install 14.20.0* 

### Install Yarn v1.x

Let's install Yarn on Ubuntu via the Debian package repository:

    curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add - 
    echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list

Update the system and install yarn: *sudo apt update && sudo apt install yarn*

Confirm that yarn is installed: *yarn --version*. For it was 1.22.19. It will not work with v2.

Create folder */opt/dspaceinst/frontend* and download the source *https://github.com/DSpace/dspace-angular/archive/refs/tags/dspace-7.3.tar.gz*

Untar archive *tar xzvf dspace-7.3.tar.gz*

Install process Manager for Node.js: *npm install --global pm2*

*cd /opt/dspaceinst/frontend/dspace-angular-7.3* and run *sudo yarn install*

Build the User Interface for Production: *yarn build:prod*

Deployment: *cp -r /opt/dspaceinst/frontend/dspace-angular-7.3/dist /dspace-ui-deploy*

  The structure of /dspace-ui-deploy:
   
           /dist
             /browser 
             /server  
           /config     (Will create later)
             /config.prod.yml (Will create later)
