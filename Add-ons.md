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
    iiif.image.server = http://localhost:8182/iiif/2/
   
Add to config.prod.yml

        mediaViewer:
         image: true
         video: false
