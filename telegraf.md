# telegraf.conf

```
[global_tags]
[agent]
  interval = "1000ms"
  round_interval = false
  metric_buffer_limit = 10000
  collection_jitter = "0s"
  flush_interval = "10s"
  flush_jitter = "5s"
  precision = "0ms"
  debug = false
  quiet = false
  logfile = ""
  hostname = ""
  omit_hostname = false
###############################################################################
#                            OUTPUT PLUGINS                                   #
###############################################################################
[[outputs.influxdb]]
  urls = ["http://192.168.188.91:8086"]
  database = "couchbase"
  retention_policy = ""
  write_consistency = "any"
  timeout = "5s"
  
###############################################################################
#                            INPUT PLUGINS                                    #
###############################################################################
[[inputs.cpu]]
  percpu = true
  totalcpu = true
  collect_cpu_time = false
  report_active = false
  fielddrop = ["time_*"]
[[inputs.disk]]  
  ignore_fs = ["tmpfs", "devtmpfs", "devfs"]
[[inputs.diskio]]
[[inputs.kernel]]
[[inputs.mem]]
[[inputs.processes]]
[[inputs.swap]]
[[inputs.system]]
[[inputs.bond]]
  host_proc = "/proc"
  bond_interfaces = ["bond0", "bond1"]
[[inputs.couchbase]]
  servers = ["http://127.0.0.1:8091"]
  interval = "30s"
[[inputs.interrupts]]
[[inputs.linux_sysctl_fs]]
[[inputs.memcached]]
[[inputs.net]]
[[inputs.netstat]]
  interval = "2000ms"
[[inputs.nstat]]
  proc_net_netstat = "/proc/net/netstat"
  proc_net_snmp = "/proc/net/snmp"
  proc_net_snmp6 = "/proc/net/snmp6"
```
