FROM        quay.io/prometheus/busybox:latest
LABEL maintainer="The Prometheus Authors <prometheus-developers@googlegroups.com>"

RUN wget https://github.com/prometheus/blackbox_exporter/releases/download/v0.15.1/blackbox_exporter-0.15.1.linux-amd64.tar.gz -O /tmp/blackbox_exporter.tar.gz && \
    tar -xz -f /tmp/blackbox_exporter.tar.gz --strip-components 1 -C /bin blackbox_exporter-0.15.1.linux-amd64/blackbox_exporter && \
    rm -rf /tmp/blackbox_exporter.tar.gz

COPY blackbox.yml       /etc/blackbox_exporter/config.yml

EXPOSE      9115
ENTRYPOINT  [ "/bin/blackbox_exporter" ]
CMD         [ "--config.file=/etc/blackbox_exporter/config.yml" ]