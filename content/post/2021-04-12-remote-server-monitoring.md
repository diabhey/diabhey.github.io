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

The stations will be deployed across the globe and we need a way to effectively monitor and alert us in real-time. Thanks to [Prometheus](https://prometheus.io/), all the heavy-lifting is done for us.  In this post, I will run through a series of steps that is required to  setup Prometheus on a linux server and then go ahead and setup [Grafana Cloud](https://grafana.com/products/cloud/) as our observability platform through which can intuitively gain insights from our data sources across the globe.

### Requirements

- [Grafana Cloud Account](https://grafana.com/docs/grafana-cloud/quickstart/)
- Access to Linux Server(Node)

### Set up Prometheus and Node Exporter

Firstly, we will need to setup the [node_exporter](https://github.com/prometheus/node_exporter) that is responsible for collecting the prometheus metrics and sending it to the grafana cloud.

```bash
# Download the node_exporter
wget https://github.com/prometheus/node_exporter/releases/download/v*/node_exporter-*.*-amd64.tar.gz
# Extract and execute
tar xvfz node_exporter-*.*-amd64.tar.gz 
cd node_exporter-*.*-amd64
./node_exporter
```

Secondly, install Prometheus on the node

```bash
# Download prometheus
wget https://github.com/prometheus/prometheus/releases/download/v*/prometheus-*.*-amd64.tar.gz
# Extract 
tar xvf prometheus-*.*-amd64.tar.gz
cd prometheus-*.*
```

Now we need to configure prometheus to publish the content to grafana cloud. For that, we need to create the `prometheus.yml` in the same directory as the prometheus binary with the following content, 

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: node
    static_configs:
    - targets: ['localhost:9100']

remote_write:
  - url: https://prometheus-us-central1.grafana.net/api/prom/push
    basic_auth:
      username: "your grafana username"
      password: "your Grafana API key"
```

Save the file and we shall come back to it shortly.  The grafana `username` and `password` need to be created in order to visualise the data on the grafana dashboard. Let's login into grafana cloud portal and create a stack

![create_grafana_stack](/home/abhi/code/personal/github/diabhey.github.io/static/images/create_grafana_stack.png)

The instance will be created shortly and an **url** will be generated through which we can access our grafana enterprise stack.

![station-stack](/home/abhi/code/personal/github/diabhey.github.io/static/images/station.png)

Copy the Prometheus **remote_write** Configuration generated and add it to the `prometheus.yml` file we had created earlier.  Save the file and exit.

 Open a terminal and navigate to the prometheus and node_exporter directory and  run the following, 

```bash
# Start prometheus
./prometheus --config.file=./prometheus.yml
# Start node_exporter
./node_exporter

```

Login into the `https://<your stack>.grafana.net` and select the data source as the prometheus instance that we had created earlier. This step is needed to visualise the data being published from the nodes. You can now create your own dashboard to monitor the metrics of your choice or import [exisitng dashboards](https://grafana.com/grafana/dashboards).

For this demo, I have imported the Node Exporter Quickstart Dashboard (*id: 13978*)

![station-dashboard](/home/abhi/code/personal/github/diabhey.github.io/static/images/dashboard.png)

There you have it. On the next monitoring and observability series, I will explain how to configure our existing setup to also monitor the docker containers running on the nodes.
