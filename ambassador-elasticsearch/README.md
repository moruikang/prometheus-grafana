### ambasador domain access log dashboard
![image]https://raw.githubusercontent.com/moruikang/prometheus-grafana/master/ambassador-elasticsearch/ambassador-dashboard.jpg

- 数据源: elasticsearch

#### fluentd 解析规则
```
<source>
  @type tail
  @log_level error
  path /var/log/containers/ambassador*.log
  pos_file /var/log/ambassador.log.pos
  <parse>
    @type regexp
    # with ambassador use_proxy_proto: false and use AWS ALB replace AWS ELB, it remote client ip will be changed ,et:(?<client_ip>[^"]*),(?<alb_ip>[^,]*)
    expression /^\{"log":"ACCESS \[(?<log_time>[^\]]*)\] \\"(?<method>[^ ]*) (?<url>[^"]*) (?<protocol>[^"]*)" (?<response_code>[^ ]*)(?<response_flags> [^ ]*) (?<bytes_received>[^ ]*) (?<bytes_sent>[^ ]*) (?<duration>[^ ]*) (?<upstream-service-time>[^]*) \\"(?<client_ip>[^"]*),(?<alb_ip>[^,]*)\\" \\"(?<user-agent>[^"]*)\\" \\"(?<request_id>[^"]*)\\" \\"(?<domain>[^"]*)\\" \\"(?<upstream_host>[^"]*)\\"/
  </parse>
  tag ambassador
</source>


<filter ambassador.**>
  @type geoip

  # Specify one or more geoip lookup field which has ip address (default: host)
  geoip_lookup_keys  client_ip

  # Specify optional geoip database (using bundled GeoLiteCity databse by default)
  # geoip_database    "/path/to/your/GeoIPCity.dat"
  # Specify optional geoip2 database (using bundled GeoLite2 database by default)
  # geoip2_database   "/path/to/your/GeoLite2-City.mmdb"
  # Specify backend library (geoip2_c, geoip, geoip2_compat)
  backend_library geoip2_c

  # Set adding field with placeholder (more than one settings are required.)
  <record>
    city            ${city.names.en["client_ip"]}
    latitude        ${location.latitude["client_ip"]}
    longitude       ${location.longitude["client_ip"]}
    country         ${country.iso_code["client_ip"]}
    country_name    ${country.names.en["client_ip"]}
    postal_code     ${postal.code["client_ip"]}
    region_code     ${subdivisions.0.iso_code["client_ip"]}
    region_name     ${subdivisions.0.names.en["client_ip"]}
  </record>

  # To avoid get stacktrace error with `[null, null]` array for elasticsearch.
  skip_adding_null_record  true
</filter>


<match ambassador>
...
</match>
```
