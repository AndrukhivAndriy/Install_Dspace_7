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
 8. 
