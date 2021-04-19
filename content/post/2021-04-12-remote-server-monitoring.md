---
title: "Remote Server Monitoring and Observability"
date : '2021-04-12'
draft: false
toc : false
backtotop : no
disable_comments : true
categories: 
    - cloud
tags:
    - monitoring
    - observability
---

The stations will be deployed across the globe and we need a way to effectively monitor and alert us in real-time. Thanks to [Prometheus](https://prometheus.io/), all the heavy-lifting is done for us.  In this post, I will run through a series of steps that is required to  setup Prometheus on a Linux server and then go ahead and setup [Grafana Cloud](https://grafana.com/products/cloud/) as our observability platform through which can intuitively gain insights from our data sources across the globe.

### Requirements

- [Grafana Cloud Account](https://grafana.com/docs/grafana-cloud/quickstart/)
- Access to Linux Server(Node)

### Set up Prometheus, Node Exporter and cAdvisor

Firstly, we will need to setup the [node_exporter](https://github.com/prometheus/node_exporter) that is responsible for collecting the prometheus metrics and sending it to the grafana cloud and then install prometheus on the node and even run them as a systemd service. [Here](https://grafana.com/docs/grafana-cloud/quickstart/agent_linuxnode/) you can find the instructions to set it up.  In case you want to run the services as docker containers, look no further.  We will also be using cAdvisor(Container Advisor) that collects the container performance metrics.

Let's open a terminal,

```bash
# Create a node_monitor directory on the node where you are running the monitoring services
mkdir ~/node_monitor && cd node_monitor
# Create a docker-compose file to setup the services
touch docker-compose.ymlinuxl
# Create a prometheus directory to store the config 
mkdir prometheus && cd prometheus
touch prometheus.yml
```

#### Configure Prometheus

Now we need to configure prometheus to publish the content to grafana cloud. For that, we need to edit the contents of the `prometheus.yml` with the following content,

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: node
    static_configs:
    - targets: ['localhost:9090','cadvisor:8080','node-exporter:9100']

remote_write:
  - url: https://prometheus-us-central1.grafana.net/api/prom/push
    basic_auth:
      username: "your grafana username"
      password: "your Grafana API key"
```

Save the file and we shall come back to it shortly.  The grafana `username` and `password` need to be created in order to visualize the data on the grafana dashboard.

- Let's login into grafana cloud portal and create a stack.

- The instance will be created shortly and an **url** will be generated through which we can access our grafana enterprise stack.

- Copy the Prometheus **remote_write** Configuration generated and add it to the `prometheus.yml` file we had created earlier.  Save the file and exit.

#### Create the monitoring services

​Add the following contents to the `docker-compose.yml`

```yml
version: '3'
services:
  prometheus:
    image: prom/prometheus:latest
    ports:
      - 9000:9090
    volumes:
      - ./prometheus:/etc/prometheus
      - prometheus-data:/prometheus
    command: --web.enable-lifecycle  --config.file=/etc/prometheus/prometheus.yml    

    expose:
      - 9090
    ports:
      - 9090:9090
    links:
      - cadvisor:cadvisor
      - node-exporter:node-exporter

  node-exporter:
    image: prom/node-exporter:latest
    container_name: monitoring_node_exporter
    restart: unless-stopped
    expose:
      - 9100

  cadvisor:
    image: google/cadvisor:latest
    container_name: monitoring_cadvisor
    restart: unless-stopped
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:rw
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
    expose:
      - 8080

volumes:
  prometheus-data:
```

Launch the services by running the following,

```bash
docker-compose -f docker-compose.yml up -d
```

At this point, we have all the necessary services in place to start visualizing the data from the nodes.

### Grafana Cloud

- Login into the `https://<your stack>.grafana.net` and select the data source as the prometheus instance that we had created earlier. This step is needed to visualise the data being published from the nodes. You can now create your own dashboard to monitor the metrics of your choice or import [exisitng dashboards](https://grafana.com/grafana/dashboards).

- For this demo, I have imported the Node Exporter Quickstart Dashboard (**id: 13978**) and also the Docker and System Monitoring Dashboard(**id: 893**)

There you have it. Feel free to configure and play with these tools as they have much more to offer.
