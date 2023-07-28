# sonnen_exporter

sonnen_exporter is a [Prometheus](https://prometheus.io/) [Exporter](https://prometheus.io/docs/instrumenting/exporters/) for batteries from German Manufacturer [Sonnen](https://sonnen.de). It's based on Docker and [prometheus-community/json_exporter](https://github.com/prometheus-community/json_exporter).

It uses API V2 of local battery, docs are available at [your local battery](http://YOURBATTERYIP/api/doc.html).

# Disclaimer

I'm not involved in any way in Sonnen GmbH and this repository is not an official Sonnen GmbH project. This is just a hobby project of some nerd. Keep in mind that there is no encryption or authentication/authorization included, so I suggest to run this only in a protected environment (like your LAN ... hopefully 😉) and don't publish this service to the Internet. If your Prometheus is not running in the same environment as this project, please use some kind of tunnel technology like VPN.

# Usage

Requirements:

* Have a server with Docker installed
* Have a Sonnen battery which supports API V2

To use sonnen_exporter, clone this repository and open [json_exporter_config.yml](/json_exporter_config.yml). Search for

`Auth-Token: 'REPLACETHISWITHYOURAUTHTOKEN'`

and insert your Auth token. Please note there are multiple occurances of this line. You can find your specific Auth token at [your local battery Web UI](http://YOURBATTERYIP/dash/software-integration/json-api). Please make sure to enable "Read API" at same page before accessing API. I also suggest to disable "Write API" and test your config using curl.

After that, sonnen_exporter can be started using

`docker compose up`

or 

`docker compose up -d` (daemonized)

in repository directory.

# How it works

This will start a web service at port 7979. Following APIs can be scraped by Prometheus:

| Description | URL |
| --- | --- |
| Overall status of entire system | http://YOURSONNENEXPORTERIP:7979/probe?module=sonnen_v2_status&target=http://YOURSONNENBATTERYIP/api/v2/status |
| Battery specific data | http://YOURSONNENEXPORTERIP:7979/probe?module=sonnen_v2_battery&target=http://YOURSONNENBATTERYIP/api/v2/battery |
| Inverter specific data | http://YOURSONNENEXPORTERIP:7979/probe?module=sonnen_v2_inverter&target=http://YOURSONNENBATTERYIP/api/v2/inverter |
| Configuration data | http://YOURSONNENEXPORTERIP:7979/probe?module=sonnen_v2_configurations&target=http://YOURSONNENBATTERYIP/api/v2/configurations |

# Further information

For more information please check [prometheus-community/json_exporter](https://github.com/prometheus-community/json_exporter) docs.

# Prometheus sample config

Your prometheus.yml which targets sonnen_exporter could look like this:

```
[...]
- job_name: sonnen_v2_status
  honor_timestamps: true
  scheme: http
  metrics_path: /probe
  params:
    module: ['sonnen_v2_status']
    target: ['http://YOURSONNENBATTERYIP/api/v2/status']
  static_configs:
  - targets:
    - YOURSONNENEXPORTERIP:7979
- job_name: sonnen_v2_battery
  honor_timestamps: true
  scheme: http
  metrics_path: /probe
  params:
    module: ['sonnen_v2_battery']
    target: ['http://YOURSONNENBATTERYIP/api/v2/battery']
  static_configs:
  - targets:
    - YOURSONNENEXPORTERIP:7979
- job_name: sonnen_v2_inverter
  honor_timestamps: true
  scheme: http
  metrics_path: /probe
  params:
    module: ['sonnen_v2_inverter']
    target: ['http://YOURSONNENBATTERYIP/api/v2/inverter']
  static_configs:
  - targets:
    - YOURSONNENEXPORTERIP:7979
- job_name: sonnen_v2_configurations
  honor_timestamps: true
  scrape_interval: 1d
  scheme: http
  metrics_path: /probe
  params:
    module: ['sonnen_v2_configurations']
    target: ['http://YOURSONNENBATTERYIP/api/v2/configurations']
  static_configs:
  - targets:
    - YOURSONNENEXPORTERIP:7979
[...]
```
