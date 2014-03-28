# fluent-plugin-geoip [![Build Status](https://travis-ci.org/y-ken/fluent-plugin-geoip.png?branch=master)](https://travis-ci.org/y-ken/fluent-plugin-geoip)

Fluentd Output plugin to add information about geographical location of IP addresses with Maxmind GeoIP databases.

fluent-plugin-geoip has bundled cost-free [GeoLite City database](http://dev.maxmind.com/geoip/legacy/geolite/) by default.<br />
Also you can use purchased [GeoIP City database](http://www.maxmind.com/en/city) ([lang:ja](http://www.maxmind.com/ja/city)) which costs starting from $50.

The accuracy details for GeoLite City (free) and GeoIP City (purchased) has described at the page below.

* http://www.maxmind.com/en/geolite_city_accuracy ([lang:ja](http://www.maxmind.com/ja/geolite_city_accuracy))
* http://www.maxmind.com/en/city_accuracy ([lang:ja](http://www.maxmind.com/ja/city_accuracy))

## Dependency

before use, install dependent library as:

```bash
# for RHEL/CentOS
$ sudo yum install geoip-devel --enablerepo=epel

# for Ubuntu/Debian
$ sudo apt-get install libgeoip-dev
```

## Installation

install with `gem` or `fluent-gem` command as:

```bash
# for fluentd
$ gem install fluent-plugin-geoip

# for td-agent
$ sudo /usr/lib64/fluent/ruby/bin/fluent-gem install fluent-plugin-geoip
```

## Usage

```xml
<match access.apache>
  type geoip

  # Specify one or more geoip lookup field which has ip address (default: host)
  # in the case of accessing nested value, delimit keys by dot like 'host.ip'.
  geoip_lookup_key  host

  # Specify geoip database (using bundled GeoLiteCity databse by default)
  geoip_database    'data/GeoLiteCity.dat'

  # Set adding field with placeholder (more than one settings are required.)
  <record>
    city            ${city['host']}
    latitude        ${latitude['host']}
    longitude       ${longitude['host']}
    country_code3   ${country_code3['host']}
    country         ${country['host']}
    country_name    ${country_name['host']}
    dma             ${dma['host']}
    area            ${area['host']}
    region          ${region['host']}
  </record>

  # Settings for tag
  remove_tag_prefix access.
  tag               geoip.${tag}

  # Set log_level for fluentd-v0.10.43 or earlier (default: warn)
  log_level         info

  # Set buffering time (default: 0s)
  flush_interval    1s
</match>
```

#### Tips: how to geolocate multiple key

```xml
<match access.apache>
  type geoip
  geoip_lookup_key  user1_host, user2_host
  user1_city        ${city['user1_host']}
  user2_city        ${city['user2_host']}
  remove_tag_prefix access.
  tag               geoip.${tag}
</match>
```

#### Advanced config samples

It is a sample to get friendly geo point recdords for elasticsearch with Yajl (JSON) parser.

```
<match input.access>
  type                   geoip
  geoip_lookup_key       host
  <record>
    # lat lon as properties
    # ex. {"lat" => 37.4192008972168, "lon" => -122.05740356445312 }
    location_properties  { "lat":${latitude['host']}, "lon":${longitude['host']}}
  
    # lat lon as string
    # ex. 37.4192008972168,-122.05740356445312
    location_string      ${latitude['host']},${longitude['host']}
    
    # lat lon as array (it is useful for Kibana's bettermap.)
    # ex. [-122.05740356445312, 37.4192008972168]
    location_array       [${longitude['host']},${latitude['host']}]
  </record>
  remove_tag_prefix      access.
  tag                    geoip.${tag}
</match>
```

## Tutorial

#### configuration

```xml
<source>
  type forward
</source>

<match test.geoip>
  type copy
  <store>
    type stdout
  </store>
  <store>
    type    geoip
    geoip_lookup_key  host
    <record>
      city  ${city['host']}
      lat   ${latitude['host']}
      lon   ${longitude['host']}
    </record>
    remove_tag_prefix test.
    tag     debug.${tag}
  </store>
</match>

<match debug.**>
  type stdout
</match>
```

#### result

```bash
# forward record with Google's ip address.
$ echo '{"host":"66.102.9.80","message":"test"}' | fluent-cat test.geoip

# check the result at stdout
$ tail /var/log/td-agent/td-agent.log
2013-08-04 16:21:32 +0900 test.geoip: {"host":"66.102.9.80","message":"test"}
2013-08-04 16:21:32 +0900 debug.geoip: {"host":"66.102.9.80","message":"test","city":"Mountain View","lat":37.4192008972168,"lon":-122.05740356445312}
```

For more details of geoip data format is described at the page below in section `GeoIP City Edition CSV Database Fields`.<br />
http://dev.maxmind.com/geoip/legacy/csv/

## Placeholders

Provides these placeholders for adding field of geolocate results.

* ${city}
* ${latitude}
* ${longitude}
* ${country_code3}
* ${country_code}
* ${country_name}
* ${dma_code}
* ${area_code}
* ${region}

## Parameters

* `include_tag_key` (default: false)
* `tag_key`

Add original tag name into filtered record using SetTagKeyMixin.<br />
Further details are written at http://docs.fluentd.org/articles/in_exec

* `remove_tag_prefix`
* `remove_tag_suffix`
* `add_tag_prefix`
* `add_tag_suffix`

Set one or more option are required unless using `tag` option for editing tag name. (HandleTagNameMixin feature)

* `tag`

On using this option with tag placeholder like `tag geoip.${tag}` (test code is available at [test_out_geoip.rb](https://github.com/y-ken/fluent-plugin-geoip/blob/master/test/plugin/test_out_geoip.rb)), it will be overwrite after these options affected. which are remove_tag_prefix, remove_tag_suffix, add_tag_prefix and add_tag_suffix.

* `flush_interval` (default: 0 sec)

Set buffering time to execute bulk lookup geoip.

## Articles

* [IPアドレスを元に位置情報をリアルタイムに付与する fluent-plugin-geoip v0.0.1をリリースしました #fluentd - Y-Ken Studio](http://y-ken.hatenablog.com/entry/fluent-plugin-geoip-has-released)<br />
http://y-ken.hatenablog.com/entry/fluent-plugin-geoip-has-released

* [初の安定版 fluent-plugin-geoip v0.0.3 をリリースしました #fluentd- Y-Ken Studio](http://y-ken.hatenablog.com/entry/fluent-plugin-geoip-v0.0.3)<br />
http://y-ken.hatenablog.com/entry/fluent-plugin-geoip-v0.0.3

* [fluent-plugin-geoip v0.0.4 をリリースしました。ElasticSearch＋Kibanaの世界地図に位置情報をプロットするために必要なFluentdの設定サンプルも紹介します- Y-Ken Studio](http://y-ken.hatenablog.com/entry/fluent-plugin-geoip-v0.0.4)<br />
http://y-ken.hatenablog.com/entry/fluent-plugin-geoip-v0.0.4

* [Released GeoIP plugin to work together with ElasticSearch + Kibana v3](https://groups.google.com/d/topic/fluentd/OVIcH_SKBwM/discussion)<br />
https://groups.google.com/d/topic/fluentd/OVIcH_SKBwM/discussion

* [Fluentd、Amazon RedshiftとTableauを用いたカジュアルなデータ可視化 | SmartNews開発者ブログ](http://developer.smartnews.be/blog/2013/10/03/easy-data-analysis-using-fluentd-redshift-and-tableau/)<br />
http://developer.smartnews.be/blog/2013/10/03/easy-data-analysis-using-fluentd-redshift-and-tableau/

## TODO

Pull requests are very welcome!!

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request

## Copyright

Copyright (c) 2013- Kentaro Yoshida ([@yoshi_ken](https://twitter.com/yoshi_ken))

## License

Apache License, Version 2.0

This product includes GeoLite data created by MaxMind, available from
<a href="http://www.maxmind.com">http://www.maxmind.com</a>.
