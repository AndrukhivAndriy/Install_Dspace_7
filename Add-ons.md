# Install image viewer for Dspace 7.4

1. Install Cantaloupe image server. Download it *wget -c  https://github.com/cantaloupe-project/cantaloupe/releases/download/v5.0.5/cantaloupe-5.0.5.zip *
2. *unzip canta\**
3. *cp cantaloupe.properties.sample cantaloupe.properties*
4. *cp delegates.rb.sample delegates.rb*
5. Make service:

           [Unit]
          Description=Cantaloupe

          [Service]
           ExecStart=java -Dcantaloupe.config=/opt/cantaloupe/cantaloupe.properties -Xmx2g \
           <------><------>-jar /opt/cantaloupe/cantaloupe-5.0.5.jar

           [Install]
           WantedBy=multi-user.target

Edit dspace.cfg by adding iiif at the the end of the string

    event.dispatcher.default.consumers = versioning, discovery, eperson, iiif

Config config/modules/iiif.cfg

    iiif.enabled = true
    iiif.image.server = https://YOURDOMAIN:8183/iiif/3
    iiif.search.url = ${solr.server}/${solr.multicorePrefix}word_highlighting
    
Add to config.prod.yml

        mediaViewer:
         image: true
         video: false

Config Cantaloupe:

            https.enabled = true^M
            https.host = YOUDOMAIN^M
            https.port = 8183^M
            https.key_store_type = JKS^M
            https.key_store_password = KEY^M
            https.key_store_path = /etc/tomcat9/private/server.keystore ^M
            https.key_password = KEY^M
            source.static = HttpSource
            HttpSource.BasicLookupStrategy.url_prefix = https://YOURDOMAIN:8443/server/api/core/bitstreams/
            HttpSource.BasicLookupStrategy.url_suffix = /content

Config Mirador (JPEG viewer on frontend)

https://github.com/DSpace/dspace-angular/blob/main/src/mirador-viewer/index.js

# MAIN

Every item must have metadatd field:

Note that the dspace.iiif.enabled metadata field MUST be added to the Item and set to "true"

Other data: https://wiki.lyrasis.org/display/DSDOC7x/IIIF+Configuration
