gogstash
========

Logstash like, written in golang

[![Build Status](https://travis-ci.org/tsaikd/gogstash.svg?branch=master)](https://travis-ci.org/tsaikd/gogstash)

* Download gogstash from github
	* [check latest version](https://github.com/tsaikd/gogstash/releases)
* Use docker image [tsaikd/gogstash](https://registry.hub.docker.com/u/tsaikd/gogstash/)

```
curl 'https://github.com/tsaikd/gogstash/releases/download/0.1.8/gogstash-Linux-x86_64' -SLo gogstash && chmod +x gogstash
```

* Configure for ubuntu-sys.json (example)
```
{
	"input": [
		{
			"type": "exec",
			"command": "sh",
			"interval": 60,
			"message_prefix": "%{@timestamp} [df] ",
			"args": ["-c", "df -B 1 / | sed 1d"]
		},
		{
			"type": "exec",
			"command": "sh",
			"interval": 60,
			"message_prefix": "%{@timestamp} [diskstat] ",
			"args": ["-c", "grep '0 [sv]da ' /proc/diskstats"]
		},
		{
			"type": "exec",
			"command": "sh",
			"interval": 60,
			"message_prefix": "%{@timestamp} [loadavg] ",
			"args": ["-c", "cat /proc/loadavg"]
		},
		{
			"type": "exec",
			"command": "sh",
			"interval": 60,
			"message_prefix": "%{@timestamp} [netdev] ",
			"args": ["-c", "grep '\\beth0:' /proc/net/dev"]
		},
		{
			"type": "exec",
			"command": "sh",
			"interval": 60,
			"message_prefix": "%{@timestamp} [meminfo]\n",
			"args": ["-c", "cat /proc/meminfo"]
		}
	],
	"output": [
		{
			"type": "report"
		},
		{
			"type": "redis",
			"key": "gogstash-ubuntu-sys-%{host}",
			"host": ["127.0.0.1:6379"]
		}
	]
}
```

* Configure for dockerstats.json (example)
```
{
	"input": [
		{
			"type": "dockerstats"
		}
	],
	"output": [
		{
			"type": "report"
		},
		{
			"type": "redis",
			"key": "gogstash-docker-%{host}",
			"host": ["127.0.0.1:6379"]
		}
	]
}
```

* Config format with YAML for dockerstats.json (example)
```
input:
  - type: dockerstats
output:
  - type: report
  - type: redis
    key: "gogstash-docker-%{host}"
    host:
      - "127.0.0.1:6379"
```

* Configure for nginx.yml with gonx filter (example)

```yml
input:
  - type: redis
    host: redis.server:6379
    key:  filebeat-nginx
    connections: 1

filter:
  - type: gonx
    format: '$clientip - $auth [$time_local] "$full_request" $response $bytes "$referer" "$agent"'
    source: message
  - type: gonx
    format: '$verb $request HTTP/$httpversion'
    source: full_request
  - type: date
    format: "02/Jan/2006:15:04:05 -0700"
    source: time_local
  - type: remove_field
    fields: ["full_request", "time_local"]
  - type: add_field
    key: host
    value: "%{beat.hostname}"
  - type: geoip2
    db_path: "GeoLite2-City.mmdb"
    ip_field: clientip
    key: req_geo
  - type: typeconv
    conv_type: int64
    fields: ["bytes", "response"]

output:
  - type: elastic
    url: "http://elastic.server:9200"
    index: "log-nginx-%{+@2006-01-02}"
    document_type: "%{type}"
```

* Run gogstash for nginx example (command line)
```
GOMAXPROCS=4 ./gogstash --CONFIG nginx.json
```

* Run gogstash for dockerstats example (docker image)
```
docker run -it --rm \
	--name gogstash \
	--hostname gogstash \
	-e GOMAXPROCS=4 \
	-v "/var/run/docker.sock:/var/run/docker.sock" \
	-v "${PWD}/dockerstats.json:/gogstash/config.json:ro" \
	tsaikd/gogstash:0.1.8
```

## Supported inputs

See [input modules](input) for more information

* [docker log](input/dockerlog)
* [docker stats](input/dockerstats)
* [exec](input/exec)
* [file](input/file)
* [http](input/http)
* [httplisten](input/httplisten)
* [redis](input/redis)
* [socket](input/socket)

## Supported filters

See [filter modules](filter) for more information

* [addfield](filter/addfield)
* [json](filter/json)

## Supported outputs

See [output modules](output) for more information

* [amqp](output/amqp)
* [elastic](output/elastic)
* [email](output/email)
* [prometheus](output/prometheus)
* [redis](output/redis)
* [report](output/report)
* [stdout](output/stdout)
