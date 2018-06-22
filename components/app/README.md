# A demo App

Since we want to demo how to get http response time through `Envoy`, 
we will deploy a demo app to simulate a http service, which will have a latency of specified milliseconds for each http request.

# Run the app as docker container

```bash
docker run -d -p 8080:8080 beekman9527/webapp
```
Let's call this http service as `video service`.



# Access the http service

```bash
curl http://localhost:8080/workload.php/?value=100
```

In this script, we provide the http request with a parameter `value=100`, which means the response time will be around `100 ms`.

# Access the http service via `Envoy`
If `Envoy` is deployed for this service as [specified here](https://github.com/songbinliu/envoyMetrics/tree/master/components/envoy), then the service can access through another port:

```bash
curl http://localhost:9090/workload.php/?value=100
``` 




