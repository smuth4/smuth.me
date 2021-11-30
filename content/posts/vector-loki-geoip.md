+++
tags = ["vector", "grafana", "loki"]
date = "2021-11-29"
title = "Tracking Geo IP information with Vector, Loki and Grafana"
+++

Normally my logging stack of choice is [ELK](https://www.elastic.co/elasticsearch/), but recently I've been digging more into [Loki](https://grafana.com/docs/loki/latest/), designed to be the logging equivalent of Prometheus. It's been very interesting dealing with a low-cardinality log system after getting so used to ELK, but one feature I've definitely been missing is the ability to easily add GeoIP from arbitrary logs. There's an [open issue](https://github.com/grafana/loki/issues/2120) for adding it, but in the mean time one comment suggested using [vector](https://vector.dev/).

In my case, the log lines I was interested looked like this:
```
1637792122 I conn_pool_manager->255.152.255.207: Handshake dropped: connection backoff.
1637792123 I conn_pool_manager->71.255.178.255: Handshake dropped: certificate rejected.
1637792126 I conn_pool_manager->94.255.255.178: Adding incoming connection: fd:2074.
```

The first step is to get them into loki with the information we need. [Vector's remap language](https://vector.dev/docs/reference/vrl/) is extremely powerful and allows us to create this configuration:

```toml
# Watch the log files
[sources.in]
type = "file"
include = [ "/var/log/connections/*.log" ]
ignore_older_secs = 600
read_from = "beginning"


# Parse log message, mainly to extract the original timestamp and IP address
[transforms.remap_conn_pool]
inputs = [ "in"]
type = "remap"
source = '''
  . |= parse_regex(.message, r'^(?P<timestamp>\d+)\W(?P<level>\w)\Wconn_pool_manager->(?P<ip_address>[\d\.]+):(?P<trailer>.*)') ??
       {"err": "could not parse"}
  .timestamp = parse_timestamp(.timestamp, "%s") ?? now()
'''

# Use a locally downloaded MaxMind DB to generate GeoIP info
[transforms.geo_ip]
type = "geoip"
inputs = [ "remap_conn_pool" ]
database = "/etc/vector/GeoLite2-City.mmdb"
source = "ip_address"
target = "geoip"

# For debugging purposes
[sinks.out]
inputs = ["geo_ip"]
type = "console"
encoding.codec = "json"

# Send output to loki
[sinks.loki]
inputs = ["geo_ip"]
type = "loki"
endpoint = "https://loki.smuth.me/"
encoding.codec = "json"
labels = {"app"= "vector"}
```

Now that the information is in Loki, we can use the [Geomap panel](https://grafana.com/docs/grafana/latest/visualizations/geomap/) to display the locations inside grafana. In this particular case, this query was all that was needed:
```
count_over_time(
  {app="vector"}
  | json geoip_longitude="geoip.longitude",geoip_latitude="geoip.latitude"
  | geoip_latitude != ""
  [$__range])
```

This gets all JSON lines from logs created by the vector config with GeoIP information, filters for non-empty values, and counts each entry for the range provided. However, in order to make it more palatable for grafana, we need to apply some transforms to the data in grafana, namely labels-to-fields, merge, and then converting the lat and long from strings to numbers. [Here](/static/grafana/geoip-panel.json) is an example panel JSON, to get started with.
