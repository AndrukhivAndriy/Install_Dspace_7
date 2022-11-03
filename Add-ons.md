# Install image viewer for Dspace 7.4

1. Install Cantaloupe image server. Download it *wget -c  https://github.com/cantaloupe-project/cantaloupe/releases/download/v5.0.5/cantaloupe-5.0.5.zip *
2. *unzip canta\**
3. *cp cantaloupe.properties.sample cantaloupe.properties*
4. *cp delegates.rb.sample delegates.rb*
5. run it *java -cp cantaloupe-5.0.5.jar -Dcantaloupe.config=cantaloupe.properties -Xmx1500m -Xms1000m edu.illinois.library.cantaloupe.StandaloneEntry SyslogIdentifier=cantaloupe* 
