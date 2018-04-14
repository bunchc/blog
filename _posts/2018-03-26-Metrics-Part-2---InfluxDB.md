---
title: "Metrics: Part 2 - InfluxDB"
date: 2018-03-26
layout: "post"
categories: metrics, netdata, influxdb, timeseries
---

The first stop in [our metrics adventure](http://blog.codybunch.com/2018/03/12/Metrics-Adventures-in-Netdata-TICK-stack-and-ELK-Stack/) was to install and configure Netdata to collect system level statistics. We then configured all of the remote Netdata agents to send all of their data to a central proxy node. This is a great start, however, it is not without some challenges. That is, the data supplied by Netdata, while extensive, needs to be stored elsewhere for more than basic reporting, analysis, and trending.

What then should we do with said data? In addition to allowing you to stream data between Netdata instances as we did in our prior post, you can also stream to various databases, both standard and specialized.

As we are exploring TICK stack, we will stream our metrics data into InfluxDB, a specialized time-series database.

## InfluxDB

InfluxDB is the "I" in TICK Stack. InfluxDB is a time series database, designed specifically for metrics and event data. This is a good thing, as we have quite an extensive set of system metrics provided by Netdata that we will want to retain so we can observe trends over time or search for anomalies.

In this post we will configure InfluxDB to receive data from Netdata. Additionally, we will reconfigure our Netdata proxy node to ship metric data to InfluxDB.

> The following sections rely heavily on the [Ansible playbooks from Larry Smith Jr.](https://github.com/mrlesmithjr?utf8=%E2%9C%93&tab=repositories&q=ansible&type=&language=) A basic understanding of Ansible is assumed

### Netdata - Configure Netdata to export metrics

Take a moment to review the configuration and metrics collection architecture from our [first post](http://blog.codybunch.com/2018/03/12/Metrics-Adventures-in-Netdata-TICK-stack-and-ELK-Stack/).

Reviewed? Good. While Netdata will allow us to ship metrics data from each installed instance of Netdata, this can be quite noisy, or not otherwise provide the control you would like. Fortunately, the configuration to send metrics data is the same in either case.

One other consideration when shipping data from Netdata to InfluxDB, is how best to take the data in. Netdata supports different data export types: graphite, opentsdb, json, and prometheus. Our environment will be configured to send data using the opentsdb ```telnet interface```.

> __Note:__ As none of these are native to InfluxDB, they are exceedingly difficult to use with InfluxDB-Relay.

To reconfigure your Netdata proxy using the [ansible-netdata](https://github.com/mrlesmithjr/ansible-netdata) role, the following playbook can be used:

```
---
- hosts: netdata-proxies
  vars:
    netdata_configure_archive: true
    netdata_archive_enabled: 'yes'
    netdata_archive_type: 'opentsdb'
    netdata_archive_destination: "{{ influxdb }}:4242"
    netdata_archive_prefix: 'netdata'
    netdata_archive_data_source: 'average'
    netdata_archive_update: 1
    netdata_archive_buffer_on_failures: 30
    netdata_archive_timeout: 20000
    netdata_archive_send_names: true
  roles:
    - role: ansible-netdata
```

The variables above tell netdata to:

* Archive data to a backend
* Enable said backend (as netdata only supports one at a time)
* Configure the opentsdb protocol to send data
* Configure the host and port to send to

Additionally, it configures some additional features:

* Send data once a second
* Keep 30 seconds of data in case of connection issues
* Send field names instead of UUID
* Specify connection timeout in milliseconds

Once this playbook has run, your netdata instance will start shipping data. Or, trying to anyways, we haven't yet installed and configured InfluxDB to capture it. Let's do that now.

### InfluxDB - Install and configure InfluxDB

As discussed above, InfluxDB is a stable, reliable, timeseries database. Tuned for storing our metrics for long term trending. For this environment we are going to install a single small node. Optimizing and scaling are a topic in and of themselves. To ease installation and maintenance, InfluxDB will be installed using the [ansible-influxdb role](https://github.com/mrlesmithjr/ansible-influxdb).

The following Ansible playbook configures the ansible-influxdb role to listen for opentsdb messages from our Netdata instance.

```
---
- hosts: influxdb
  vars:
    influxdb_config: true
    influxdb_version: 1.5.1
    influxdb_admin:
      enabled: true
      bind_address: "0.0.0.0"
    influxdb_opentsdb:
      database: netdata
      enabled: true
    influxdb_databases:
      - host: localhost
        name: netdata
        state: present
  roles:
    - role: ansible-influxdb
```

A quick breakdown of the settings supplied:

* Configure influxdb rather than use the default config
* Use influxdb 1.5.1
* Enable the admin interface
* Have InfluxDB listen for opentsdb messages and store them in the ```netdata``` database
* Create the netdata database

After this playbook run is successful, you will have an instance of InfluxDB collecting stats from your Netdata proxy!

## Did it work?

If both playbooks ran successfully, system metrics will be flowing something like this:

```
nodes ==> netdata-proxy ==> influxdb
```

You can confirm this by logging into your InfluxDB node and running the following commands:

Check that InfluxDB is running:

```
# systemctl status influxdb
● influxdb.service - InfluxDB is an open-source, distributed, time series database
   Loaded: loaded (/lib/systemd/system/influxdb.service; enabled; vendor preset: enabled)
   Active: active (running) since Sat 2018-04-14 18:29:03 UTC; 9min ago
     Docs: https://docs.influxdata.com/influxdb/
 Main PID: 11770 (influxd)
   CGroup: /system.slice/influxdb.service
           └─11770 /usr/bin/influxd -config /etc/influxdb/influxdb.conf
```

Check that the ```netdata``` database was created:

```
# influx
Connected to http://localhost:8086 version 1.5.1
InfluxDB shell version: 1.5.1
> SHOW DATABASES;
name: databases
name
----
netdata
_internal
```

Check that the ```netdata``` database is receiving data:

```
Connected to http://localhost:8086 version 1.5.1
InfluxDB shell version: 1.5.1
> use netdata;
Using database netdata
> show series;
key
---
netdata.apps.cpu.apps.plugin,host=netdata-01
netdata.apps.cpu.build,host=netdata-01
netdata.apps.cpu.charts.d.plugin,host=netdata-01
netdata.apps.cpu.cron,host=netdata-01
netdata.apps.cpu.dhcp,host=netdata-01
netdata.apps.cpu.kernel,host=netdata-01
netdata.apps.cpu.ksmd,host=netdata-01
netdata.apps.cpu.logs,host=netdata-01
netdata.apps.cpu.netdata,host=netdata-01
netdata.apps.cpu.nfs,host=netdata-01
netdata.apps.cpu.other,host=netdata-01
netdata.apps.cpu.puma,host=netdata-01
netdata.apps.cpu.python.d.plugin,host=netdata-01
netdata.apps.cpu.ssh,host=netdata-01
netdata.apps.cpu.system,host=netdata-01
netdata.apps.cpu.tc_qos_helper,host=netdata-01
netdata.apps.cpu.time,host=netdata-01
netdata.apps.cpu_system.apps.plugin,host=netdata-01
```

# Summary

With that, you now have high resolution system metrics being collected and sent to InfluxDB for longer term storage, analysis, and more.