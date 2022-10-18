# Prometheus v2.39.1 with Dspace 7.4

1. *apt update && apt upgrade -y*
2. *cd /opt*
3. *wget -c https://github.com/prometheus/prometheus/releases/download/v2.39.1/prometheus-2.39.1.linux-amd64.tar.gz*
4. *tar xvfz prometheus-*.tar.gz*
5. create a configuration file directory: *mkdir -p /etc/prometheus*
6. create the data directory *mkdir -p /var/lib/prometheus*
7. *mv prometheus promtool /usr/local/bin/*
8. Move the template configuration file prometheus.yml to /etc/prometheus: *mv prometheus.yml /etc/prometheus/prometheus.yml*
9. We have installed Prometheus. Let’s verify the version: *prometheus --version

## Create Prometheus System Group

1. *groupadd --system prometheus*
2. *useradd -s /sbin/nologin --system -g prometheus prometheus*
3. *chown -R prometheus:prometheus /etc/prometheus/  /var/lib/prometheus/*
4. *chmod -R 775 /etc/prometheus/ /var/lib/prometheus/*
5. create a systemd service:

        [Unit]
        Description=Prometheus
        Wants=network-online.target
        After=network-online.target

        [Service]
        User=prometheus
        Group=prometheus
        Restart=always
        Type=simple
        ExecStart=/usr/local/bin/prometheus \
            --config.file=/etc/prometheus/prometheus.yml \
            --storage.tsdb.path=/var/lib/prometheus/ \
            --web.console.templates=/etc/prometheus/consoles \
            --web.console.libraries=/etc/prometheus/console_libraries \
            --web.listen-address=0.0.0.0:9090

        [Install]
        WantedBy=multi-user.target
        
 6. *systemctl start prometheus*
 7. *systemctl enable prometheus*
 
 ## Install Node Exporter
 
 1.  let's create a system user for Node Exporter
                         
                         sudo useradd \
                          --system \
                          --no-create-home \
                          --shell /bin/false node_exporter

 2. *wget -c https://github.com/prometheus/node_exporter/releases/download/v1.4.0/node_exporter-1.4.0.linux-amd64.tar.gz*
 3. *tar -xvf node_export\**
 4. Move binary to the /usr/local/bin
 5. Verify that you can run: *node_exporter --version*
 6. create similar systemd unit file:

                [Unit]
                Description=Node Exporter
                Wants=network-online.target
                After=network-online.target

                StartLimitIntervalSec=500
                StartLimitBurst=5

                [Service]
                User=node_exporter
                Group=node_exporter
                Type=simple
                Restart=on-failure
                RestartSec=5s
                ExecStart=/usr/local/bin/node_exporter \
                        --collector.logind

                [Install]
                WantedBy=multi-user.target
                
7. To automatically start the Node Exporter after reboot: *systemctl enable node_exporter*
8. *systemctl start node_exporter*
9. create a static target in /etc/prometheus/prometheus.yml:

                - job_name: node_export
                 static_configs:
                   - targets: ["localhost:9100"]

10. Before, restarting check if the config is valid: *promtool check config /etc/prometheus/prometheus.yml*
11. *service prometheus restart*

## Install Grafana

1. Install all the dependencies: *apt-get install -y apt-transport-https software-properties-common*
2. add GPG key: *wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -*
3. Add this repository: *echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list*
4. *apt update*
5. *apt install grafana*
6. *systemctl enable grafana-server*
7. *systemctl start grafana-server*
8. *ufw allow 3000*
9. Config Grafana. Enable https and domain. Edit /etc/grafana/grafana.ini. And restart Grafana:

                protocol = https
                cert_key = /etc/grafana/KEY.key
                cert_file = /etc/grafana/sslchain.crt
                domain = yourdomain.com
                
10. Go to the browser and type: *https://yourdomain.com:3000*. The username is *admin*, and the password is *admin* as well. Change password to new one. 
11. Create a new /etc/grafana/provisioning/datasources/datasources.yaml:

                apiVersion: 1

                datasources:
                 - name: Prometheus
                  type: prometheus
                  url: http://localhost:9090
                  isDefault: true

12. *systemctl restart grafana-server*

## Install solr-exporter

1.                      sudo useradd \
                          --system \
                          --no-create-home \
                          --shell /bin/false solr_exporter
2. Find exporter in your Solr directory *opt/solr/contrib/prometheus-exporter*
3. create similar systemd unit file:

                [Unit]
                Description=Nginx Exporter
                Wants=network-online.target
                After=network-online.target

                StartLimitIntervalSec=500
                StartLimitBurst=5

                [Service]
                User=solr_exporter
                Group=solr_exporter
                Type=simple
                Restart=on-failure
                RestartSec=5s
                ExecStart=/opt/solr/contrib/prometheus-exporter/bin/solr-exporter \
                <------><------><------>     -p 9854 \
                <------><------><------>     -b http://localhost:8983/solr \
                <------><------><------>     -f /opt/solr/contrib/prometheus-exporter/conf/solr-exporter-config.xml \
                <------><------><------>     -n 8

                [Install]
                WantedBy=multi-user.target
                
4. Add a static target in /etc/prometheus/prometheus.yml:

                - job_name: 'solr'
                static_configs:
                - targets: ['localhost:9854']
 
5. Before, restarting check if the config is valid: *promtool check config /etc/prometheus/prometheus.yml*
6. Restart prometheus
7. import .json grafana dashboard from */opt/solr/contrib/prometheus-exporter/conf* . Find JETTY METRICS

## Install Postgresql-exporter

1. *wget -c https://github.com/prometheus-community/postgres_exporter/releases/download/v0.11.1/postgres_exporter-0.11.1.linux-amd64.tar.gz*
2. *tar -xzvf postgres\** 
3. Move binary to the /usr/local/bin
4.                      sudo useradd \
                           --system \
                           --no-create-home \
                           --shell /bin/false postgres_exporter
5. create similar systemd unit file in /etc/system/systemd:

                [Unit]
                Description=Prometheus exporter for Postgresql
                Wants=network-online.target
                After=network-online.target
                [Service]
                User=postgres_exporter
                Group=postgres_exporter
                WorkingDirectory=/opt/postgres_exporter
                EnvironmentFile=/opt/postgres_exporter/postgres_exporter.env
                ExecStart=/usr/local/bin/postgres_exporter --web.listen-address=:9187 --web.telemetry-path=/metrics
                Restart=always
                [Install]
                WantedBy=multi-user.target

6. Create folder */opt/postgres_exporter*
7. create */opt/postgres_exporter/postgres_exporter.env* with *DATA_SOURCE_NAME="postgresql://dspace:password@localhost:5432/dspace?sslmode=disable"*                 
8. To automatically start the Posgres Exporter after reboot: *systemctl enable postgres_exporter*
9. *systemctl start postgres_exporter*
10. create a static target in /etc/prometheus/prometheus.yml:

                - job_name: postgres
                  static_configs:
                   - targets: ["localhost:9187"]

10. Before, restarting check if the config is valid: *promtool check config /etc/prometheus/prometheus.yml*
11. *service prometheus restart*

## Install Tomcat exporter (requires Java >= 7)

1. Download exporter: *wget -c [jmx_prometheus_javaagent](https://repo1.maven.org/maven2/io/prometheus/jmx/jmx_prometheus_javaagent/0.17.2/jmx_prometheus_javaagent-0.17.2.jar)*
2. Add to */usr/share/tomcat9/bin/catalina.sh* string *CATALINA_OPTS="-javaagent:/usr/share/tomcat9/bin/jmx_prometheus_javaagent-0.17.2.jar=39081:/usr/share/tomcat9/bin/tomcat.yml"* 
3. Create file */usr/share/tomcat9/bin/tomcat.yml* with:

                ---
                startDelaySeconds: 0
                ssl: false
                LowercaseOutputName: false
                LowercaseOutputLabelNames: false


**NOTE** More info (configuration) is avaible on https://github.com/prometheus/jmx_exporter

4. Metrics will now be accessible at http://localhost:39081/metrics.
5. Add to Prometheus:

                - job_name: java
                  static_configs:
                   - targets: ["localhost:39081"]
6. Add dashboard to grafana from https://grafana.com/grafana/dashboards/8563-jvm-dashboard/

## Simple NGINX monitoring

1. wget -c https://github.com/nginxinc/nginx-prometheus-exporter/releases/download/v0.11.0/nginx-prometheus-exporter_0.11.0_linux_amd64.tar.gz
2. Modify nginx config file by adding:

                location = /nginx_stats {
                    allow ADD_EXTERNAL_IP;
                    stub_status;
                    deny all;
3. tar vxzf nginx*
4. copy *nginx-prometheus-exporter* to */usr/local/bin*

5.                          sudo useradd \
                           --system \
                           --no-create-home \
                           --shell /bin/false nginx_exporter
 
6.  Create service:

        [Unit]
        Description= Nginx Prometheus
        Wants=network-online.target
        After=network-online.target

        [Service]
        User=nginx_exporter
        Group=nginx_exporter
        Restart=always
        Type=simple
        ExecStart=/usr/local/bin/nginx-prometheus-exporter \
            -nginx.scrape-uri=https://YOUR_ADDR/nginx_stats

        [Install]
        WantedBy=multi-user.target
                     
7. Add to Prometheus:

                - job_name: nginx
                  static_configs:
                   - targets: ["localhost:9113"]    

8. Add dashboard https://raw.githubusercontent.com/nginxinc/nginx-prometheus-exporter/main/grafana/dashboard.json

## Install and config Alertmanager

1. *wget -c [https://github.com/prometheus/alertmanager/](https://github.com/prometheus/alertmanager/releases/download/v0.24.0/alertmanager-0.24.0.linux-amd64.tar.gz)*
2. *tar -xzvf alert\**
3. *useradd -M -r -s /bin/false alertmanager*
4. copy *alertmanager* to */usr/local/bin/*
5. *chown alertmanager:alertmanager /usr/local/bin/alertmanager*
6. *mkdir -p /etc/alertmanager*
7. copy alertmanager.yml to */etc/alertmanager*
8. *chown -R alertmanager:alertmanager /etc/alertmanager*
9. *mkdir -p /var/lib/alertmanager*
10. *chown alertmanager:alertmanager /var/lib/alertmanager*
11. Create a systemd unit:

                [Unit]
                Description=Prometheus Alertmanager
                Wants=network-online.target
                After=network-online.target

                [Service]
                User=alertmanager
                Group=alertmanager
                Type=simple
                ExecStart=/usr/local/bin/alertmanager \
                --config.file /etc/alertmanager/alertmanager.yml \
                --storage.path /var/lib/alertmanager

                [Install]
                WantedBy=multi-user.target
                
12. *systemctl start alertmanager*
13. *systemctl enable alertmanager*                
