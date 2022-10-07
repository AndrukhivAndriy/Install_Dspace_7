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

## Install Apache Solr (manual and as service)

We are looking for v 8.11 or above. NOT v9

---------------------1. manual ----------------------

Download solr: *wget -c https://downloads.apache.org/lucene/solr/8.11.2/solr-8.11.2.tgz* to */opt*

Rename to */opt/solr* and start via not root user: *su -c "/opt/solr/bin/solr start" - username*

By default, the script extracts the distribution archive into /opt, configures Solr to write files into /var/solr, and runs Solr as the solr user.

-------------------------2. as service. Not working well  ----------------------

Download solr: *wget -c https://downloads.apache.org/lucene/solr/8.11.2/solr-8.11.2.tgz*

Run: *tar xzf solr-8.11.2.tgz solr-8.11.2/bin/install_solr_service.sh --strip-components=2*

Then run: *bash ./install_solr_service.sh solr-8.11.2.tgz*

This command will install Solr as service. You can control Solr via command *service solr start|stop|restart*

To remove solr:

1. sudo service solr stop
2. sudo rm -r /var/solr
3. sudo rm -r /opt/solr-8.11.2
4. sudo rm -r /opt/solr
5. sudo rm /etc/init.d/solr
6. sudo deluser --remove-home solr
7. sudo deluser --group solr
8. sudo update-rc.d -f solr remove
9. sudo rm -rf /etc/default/solr.in.sh

 ## Install Apache Tomcat v9
 
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

### Restore this database on new server:

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


Create a "config.prod.yml" at /dspace-ui-deploy/config/config.prod.yml:

       ui:
        ssl: false
        host: localhost
        port: 4000
        nameSpace: /
        
      rest:
       ssl: false
       host: localhost
       port: 8080
       nameSpace: /server
       
 Let's start UI via PM2. Create file dspace-ui-deploy/dspace-ui.json :
 
       {
    "apps": [
        {
           "name": "dspace-ui",
           "cwd": "/dspace-ui-deploy",
           "script": "dist/server/main.js",
           "env": {
              "NODE_ENV": "production"
           }
         }
           ]
      }
      
Start the application using PM2: *pm2 start dspace-ui.json*

Stop the application using PM2: *pm2 stop dspace-ui.json*

Run your browser and type *http://localhost:4000* . Dspace installed!!

## Config different fitches

Config Tomcat9 SSL

1. Put your certs (.srt and .key files) in */etc/tomcat9/private*
2. *openssl pkcs12 -export -in /etc/tomcat9/private/sslchain.crt -inkey /etc/tomcat9/private/key.key -out /etc/tomcat9/private/server.p12 -name tomcat -CAfile /etc/tomcat9/private/certificate.pem -caname root*  ------ and type *yourpassword*
3. *keytool -importkeystore -deststorepass yourpassword -destkeypass yourpassword -destkeystore /etc/tomcat9/private/server.keystore -srckeystore /etc/tomcat9/private/server.p12 -srcstoretype PKCS12 -srcstorepass yourpassword -alias tomcat*
4. Add this to */etc/tomcat9/server.xml* and comment *<Connector port=8080...*:
       
       <Connector port="8443"
          minSpareThreads="25"
          maxConnections = "1024"
          acceptCount="2048"
          enableLookups="false"
          connectionTimeout="20000"
          URIEncoding="UTF-8"
          disableUploadTimeout="true"
          scheme="https" secure="true" clientAuth="false" sslProtocol="TLS"
          keystoreFile="/etc/tomcat9/private/server.keystore"
          SSLEnabled="true" keystorePass="yourpassword" />
  
Now, our backend server will work on port 8443 so you can check it typing in browser https://youdomain:8443/server . It's not a problem, becouse later we will do redirect via nginx.

## Config NGINX as proxy

Add file to */etc/nginx/sites-enabled/donai.conf* with:

 server {
 listen 443 ssl default_server;
 listen [::]:443 ssl default_server;
 access_log /var/log/nginx/reverse-access.log;
 error_log /var/log/nginx/reverse-error.log;
 root /var/www/html;
 ssl_certificate "/etc/ssl/private/sslchain.crt";
 ssl_certificate_key "/etc/ssl/private/key.key";
 location / {
  proxy_pass http://localhost:4000;
  }
 location /server {
 proxy_pass https://yourdomain:8443/server;
 }

Note: You should disable gzip for SSL traffic. See: https://bugs.debian.org/773332

## Config UI connector:

Make changes to */dspace-ui-deploy/config/config.prod.yml*:

    rest:
       ssl: true
       host: yourdomain
       port: 8443
       nameSpace: /server

Restart Tomcat: *service tomcat9 restart*

Restart UI: *cd /dspace-ui-deploy* and type *pm2 restart dspace-ui.json*

Reindex all content: */dspace/dspace index-discovery -b*

Restart Nginx: *service nginx restart*

To start Nginx automaticly after reboot: *update-rc.d nginx enable* or *systemctl enable nginx*

# Interface customization

## Languages

To delete language from lang list on navbar - go to the *front-end-source/src/config* and edit file *default-app-config.ts*. Find block *languages: LangConfig[]* and change *active* value to false for langs which you dont want to see.

If you want to add lang - you have to add this info to file *default-app-config.ts* and add file *uk.json5* to *front-end-source/src/assets*

To recompile user interface - go to *front-end-source* and run *yarn install* and then *yarn build:prod*. Copy */dist* to *dspace-ui-deploy*. 

## Change banner (background image) and logo

Main banner is here: *dspace-angular-dspace-7.3/src/themes/dspace/assets/images*. Replace for own

Main logo is here: *dspace-angular-dspace-7.3/src/assets/images* . Replace for own.

## Change main text on banner

Modify file: *dspace-angular-dspace-7.3/src/themes/dspace/app/home-page/home-news/home-news.component.html*. Also you can modify there name of background image. 

Modify file: *dspace-angular-dspace-7.3/src/index.html* and *index.csr.html*

## Change / translate text on cookie message

Add text to *dspace-angular-dspace-7.3/src/app/shared/cookies/klaro-configuration.ts* to block *translate*:
------ To disable this popup window - change in block translation value *en* to *uk*  --------
  
    uk: {
      acceptAll: "Прийняти все",
      acceptSelected: "Прийняти вибране",
      app: {
        optOut: {
          description: "Цей додаток був завантажений по замовчуванню. Але Ви можете відмовитись від нього",
          title: "(відмовитись)"
        },
        purpose: "ціль",
        purposes: "цілі",
        required: {
          description: "Цей додаток завжди потрібний",
          title: "(завжди потрібний)"
        }
      },
      close: "Закрити",
      decline: "Відхилити",
      ok: "Погоджуюсь",
      changeDescription: 'cookies.consent.update',
      consentNotice: {
        description: "Ми збираємо та обробляємо вашу особисту інформацію для таких цілей: автентифікація, налаштування та статистика",     
        learnMore: "Налаштувати"
      },
      consentModal: {
        description: "Тут ви можете переглянути та налаштувати збір інформації про Вас.",
        privacyPolicy: {
          name: "політика приватності",
          text: "Детальніше - внизу сторінки Ви знайдете покликання",
        },
        title: "Інформація, яку ми збираємо"
      },

## To change page title from DSpace Angular :: Home 

To change page title from *DSpace Angular :: Home* to your own - change *dspace-angular-dspace-7.3/cypress/integration/homepage.spec.ts*. Find string 
*cy.title().should('eq', 'DSpace Angular :: Home');* and change for your own.

# Troubleshooting

1. If you cann't upload/add file(bitstream) to iteam :

- *chown -R tomcat:tomcat /dspace*
- change */lib/systemd/system/tomcat9.service* by adding string *ReadWritePaths=/dspace/* to Security block. 
- *systemctl daemon-reload*
- service tomcat9 restart

2. Autostart Dspace

1. Create shell script to restart all related with Dspace services. Put it */etc/init.d*  neme it dspace_auto_start.sh:

  #!/bin/bash

  su -c "/opt/solr/bin/solr stop -all" - root
  su -c "/opt/solr/bin/solr start -force" - root
  sleep 10
  su -c "service tomcat9 restart" - root
  sleep 5
  #sudo /root/.nvm/versions/node/v14.20.0/bin/pm2 restart /dspace-ui-deploy/dspace-ui.json
  cd /dspace-ui-deploy
  sudo node ./dist/server/main.js

2. Create service for script - */etc/systemd/system/dspace.service*:

  [Unit]
  Description=Auto start Dspace

  [Service]
  ExecStart=/etc/init.d/dspace_auto_start.sh

  [Install]
  WantedBy=multi-user.target
  
3. *service dspace enable*

Now, if you want restart Dspace - just type *service dspace restart*

*NOTE* If you start Dspace via *node ./dist/server/main.js* - you can not stop it via PM2. You must find PID of node and kill it. Just type:

- lsof -i :4000
- kill -9 PID

--------------------------------------------------------------------------------------------------------

## Dump Dspace to Google Disk (if you have corporate account with no limits. Not for single with 2Gb GD)

  GOOGLE DRIVE

1. Download utility Gdrive : *wget https://raw.githubusercontent.com/AnimMouse/gdrive-binaries/master/linux/gdrive-linux-x64*
2. move to */usr/sbin* and rename as *gdrive*
3. type *gdrive upload ./add-shell* - you will see authentication link. Copy it and run in your browser. Paste verification code in shell.  
4. ---- All info about utility - https://github.com/prasmussen/gdrive/blob/master/README.md  -----
5. Create new folder in your Google Disk. At the of URL in browser - you can find folder Id
6. To upload file/s to Google Drive just run *gdrive upload --parent FOLDER_ID /file/to/upload*

  SHELL SCRIPT to make dumps
  
1. Create .pgpass file with content
    host:5432:somedb:someuser:somepass
2. set the permissions
    sudo chmod 600 .pgpass
3. Set the file owner as the same user using which you logged in :
    sudo chown postgres:postgres .pgpass
4. Set PGPASSFILE environment variable :
    export PGPASSFILE='/opt/.pgpass'

The hole script:
  rm -rf /home/backup/*
  sleep 10
  tar cvzf - /dspace/assetstore/ | split --bytes=2000MB - /home/backup/assetstore-$(date +%F).tar.gz
  sleep 20
  export PGPASSFILE='/opt/.pgpass'
  pg_dump -h localhost -U dspace dspace > /home/backup/postgresql-$(date +%F).sql
  sleep 20
  drive upload --parent FOLDER_ID -f /home/backup
