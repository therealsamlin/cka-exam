# Monitoring

`metrics-server` replaced `heapster`, a slim down version of `heapster`

* metrics-server is an in-memory solution
* kubelet runs cAdvisor and expose it via the kubelet API for metrics-server to consume

To view node resource usage

`kubectl top node` 

To view pod resource usage

`kubectl top pods`

# Logging

To view logs of pod

`kubectl logs -f pod`

To view logs of container in pod

`kubectl logs -f pod -c container`

