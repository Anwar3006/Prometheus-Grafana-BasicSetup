# Example Prometheus Monitoring

## Goal

Setup monitoring with [Prometheus](https://prometheus.io) and [Grafana](https://grafana.com).

## Steps

1. Run sample server: `npm install` and `node server`
2. Run Prometheus: see below
3. Visit your running Prometheus and run queries
4. Run Grafana: see below
5. Add Prometheus data source (Url: `http://localhost:9090`)
6. Import `grafana-dashboard.json` dashboard
7. Create your own dashboard from the Prometheus queries

## Requirements

- Docker
- Node Server

### Run

Create a `docker-compose.yml` file and run with:

Modify: `prometheus.yml`, go to `static_configs: targets` and create and array, replace `***.***.***.**` with your own host machine's IP or pick from your OS below:

- Host machine IP address: `ifconfig | grep 'inet 192'| awk '{ print $2}'`
- Target configuration for each OS with docker :

  - Linux: - targets: ["localhost:9090"]
  - Mac: - targets: ["docker.for.mac.host.internal:9090"]
  - Windows: - targets: ["docker.for.win.localhost:9090"]

Modify `prometheus-datasource.yml`, go to url:

- URL configuration for each OS with docker :

  - Linux: `url: http://localhost:9090`
  - Mac: `url: http://docker.for.mac.host.internal:9090`
  - Windows: `url: http://docker.for.win.localhost:9090`

```sh
$ docker up
```

To copy provisioning yaml file from local machine to grafana datasources directory

```bash
$ docker cp path/to/your/config.yaml <grafana_container_id>:/etc/grafana/provisioning/datasources/

```

Open Prometheus: [http://localhost:9090](http://localhost:9090/graph)

### Example Queries

#### Throughput

#### Error rate

Range[0,1]: number of 5xx requests / total number of requests

```
sum(increase(http_request_duration_ms_count{code=~"^5..$"}[1m])) /  sum(increase(http_request_duration_ms_count[1m]))
```

##### Request Per Minute

```
sum(rate(http_request_duration_ms_count[1m])) by (service, route, method, code)  * 60
```

#### Response Time

#### Apdex

[Apdex](https://en.wikipedia.org/wiki/Apdex) score approximation:  
`100ms` target and `300ms` tolerated response time

```
(
  sum(rate(http_request_duration_ms_bucket{le="100"}[1m])) by (service)
+
  sum(rate(http_request_duration_ms_bucket{le="300"}[1m])) by (service)
) / 2 / sum(rate(http_request_duration_ms_count[1m])) by (service)
```

> Note that we divide the sum of both buckets. The reason is that the histogram buckets are cumulative. The le="100" bucket is also contained in the le="300" bucket; dividing it by 2 corrects for that. - [Prometheus docs](https://prometheus.io/docs/practices/histograms/#apdex-score)

##### 95th Response Time

```
histogram_quantile(0.95, sum(rate(http_request_duration_ms_bucket[1m])) by (le, service, route, method))
```

##### Median Response Time:

```
histogram_quantile(0.5, sum(rate(http_request_duration_ms_bucket[1m])) by (le, service, route, method))
```

##### Average Response Time

```
avg(rate(http_request_duration_ms_sum[1m]) / rate(http_request_duration_ms_count[1m])) by (service, route, method, code)
```

#### Memory Usage

##### Average Memory Usage

In Megabyte.

```
avg(nodejs_external_memory_bytes / 1024 / 1024) by (service)
```

### Reload config

Necessary when you modified prometheus-data.

```sh
curl -X POST http://localhost:9090/-/reload
```

## Grafana

### Run

```sh
docker run -d -p 3000:3000 --name=grafana grafana/grafana-enterprise
```

[Open Grafana: http://localhost:3000](http://localhost:3000)

```
Username: admin
Password: admin
```

### Setting dashboard

- Navigate to this page for how to set up your dashboard: [Grafana Visualization Dashboard for NodeJs App](https://dev.to/ziggornif/monitoring-a-nodejs-typescript-application-with-prometheus-and-grafana-43j2)
