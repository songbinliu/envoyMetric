# envoyMetric
In this project, We will demo how to expose the http response time in [Prometheus] format through [Envoy](https://www.envoyproxy.io).

This demo has three components: [App](https://github.com/songbinliu/envoyMetrics/tree/master/components/app), [Envoy](https://github.com/songbinliu/envoyMetrics/tree/master/components/envoy) and [statsd_exporter](https://github.com/songbinliu/envoyMetrics/tree/master/components/statsd_exporter). These three components will deployed in three [Docker](https://docs.docker.com) containers.

How to deploy this demo in Kubernetes can be found [here](https://github.com/songbinliu/envoyMetrics).

# Overview of the deployment of the demo
<img width="567" alt="envoy-metrics" src="https://user-images.githubusercontent.com/27221807/32904808-3db96434-cac6-11e7-8ed6-231f6166295c.png">

## App
[This App](https://github.com/songbinliu/envoyMetrics/tree/master/components/app) will simulate a http service, which will have a latency of specified milliseconds for each http request.

Start this App in Docker container:
```bash
cd components/app
sh run.sh
```
If the app works, it can be accessed via `http://localhost:8080/workload.php/?value=100`.

## Envoy
[Envoy](https://github.com/songbinliu/envoyMetrics/tree/master/components/envoy) will work as a http proxy for the App: All http requests for the App will go through `Envoy`, and `Envoy` will monitor the http response time for the requests.

Before start the `Envoy`, provide a [correct configuration](https://github.com/songbinliu/envoyMetrics/tree/master/components/envoy/conf) file for `Envoy` image:
```bash
cd components/envoy
vim ./conf/envoy.json  ## modify this file according to your host IP address
sh run.sh
```

If everything works fine, the app can be accessed via `http://localhost:9090/workload.php/?value=100`.

In addition, some metrics (not including the response time) can be accessed via `http://localhost:8001/stats`.


## Statsd_exporter
The timer metris of `Envoy` are only forwarded to `statsd` server (in this demo, it is `statsd_exporter`).
Two functions of [`statsd_exporter`](https://github.com/prometheus/statsd_exporter) in this demo:
* Receive all the metrics (including the response time);
* Convert the metrics from `statsd` format into [`Prometheus`](https://prometheus.io) format.

Start `statsd_exporter` in Docker container:
```bash
cd components/statsd_exporter
sh run.sh
```

If everything works fine, the metrics can be accessed via `http://localhost:9102/metrics`.

Make sure *envoy_cluster* related metrics can be found in the results of `http://localhost:9102/metrics`. If not, check the two settings in `Envoy` configuration file:
* [address of statsd_exporter](https://github.com/songbinliu/envoyMetrics/blob/a73841bc562b5255a136baca7d3585764c50a601/components/envoy/conf/envoy.json#L42)
* [address of App](https://github.com/songbinliu/envoyMetrics/blob/a73841bc562b5255a136baca7d3585764c50a601/components/envoy/conf/envoy.json#L52)

Replace the two IP addresses to a non-loopback address.


### Sample metrics from statsd_exporter

```terminal
...
envoy_cluster_manager_total_clusters 1
# HELP envoy_cluster_video_service_external_upstream_rq_200 Metric autogenerated by statsd_exporter.
# TYPE envoy_cluster_video_service_external_upstream_rq_200 counter
envoy_cluster_video_service_external_upstream_rq_200 1672
# HELP envoy_cluster_video_service_external_upstream_rq_2xx Metric autogenerated by statsd_exporter.
# TYPE envoy_cluster_video_service_external_upstream_rq_2xx counter
envoy_cluster_video_service_external_upstream_rq_2xx 1672
# HELP envoy_cluster_video_service_external_upstream_rq_time Metric autogenerated by statsd_exporter.
# TYPE envoy_cluster_video_service_external_upstream_rq_time summary
envoy_cluster_video_service_external_upstream_rq_time{quantile="0.5"} 104
envoy_cluster_video_service_external_upstream_rq_time{quantile="0.9"} 106
envoy_cluster_video_service_external_upstream_rq_time{quantile="0.99"} 109
envoy_cluster_video_service_external_upstream_rq_time_sum 173175
envoy_cluster_video_service_external_upstream_rq_time_count 1673
...
```




