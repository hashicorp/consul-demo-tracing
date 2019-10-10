# Zipkin tracing with Consul Service Mesh
This demo consists of three services Ingress (HTTP), Web (HTTP), and API (gRPC)  which are configured to communicate using Consul Service Mesh. 

```
ingress (HTTP) --
                  web (HTTP) --
                                api (gRPC, 20% error rate)
```

Tracing has been configured for both the application instances and Envoy proxy using the Zipkin protocol, the spans 
will be collected by the bundled Jaeger instance.

## Running the Demo
The demo can be started using Docker Compose with the following command:

```
consul-demo-tracing/jaeger on  master [?] via 🐹 v1.13.1 on 🐳 v19.03.2
➜ docker-compose up
Creating network "jaeger_vpcbr" with driver "bridge"
Creating jaeger_ingress_1   ... done
Creating jaeger_web_1       ... done
Creating jaeger_consul_1        ... done
Creating jaeger_jaeger_1        ... done
Creating jaeger_api_1     ... done
Creating jaeger_api_proxy_1     ... done
Creating jaeger_web_proxy_1     ... done
Creating jaeger_ingress_envoy_1 ... done
Attaching to jaeger_api_1, jaeger_web_1, jaeger_ingress_1, jaeger_consul_1, jaeger_jaeger_1, jaeger_api_proxy_1, jaeger_web_proxy_1, jaeger_ingress_envoy_1
jaeger_1         | 2019/10/10 20:03:35 maxprocs: Leaving GOMAXPROCS=4: CPU quota undefined
ingress_1        | 2019-10-10T20:03:35.746Z [INFO]  Starting service: name=Ingress upstreamURIs=http://localhost:9091 upstreamWorkers=1 listenAddress=0.0.0.0:9090 service type=http
ingress_1        | 2019-10-10T20:03:35.746Z [INFO]  Adding handler for UI static files
api_1            | 2019-10-10T20:03:35.397Z [INFO]  Starting service: name=API upstreamURIs= upstreamWorkers=1 listenAddress=0.0.0.0:9090 service type=grpc
web_proxy_1      | Error retrieving members: Get http://10.5.0.2:8500/v1/agent/members?segment=_all: dial tcp 10.5.0.2:8500: connect: connection refused
web_1            | 2019-10-10T20:03:35.460Z [INFO]  Starting service: name=Web upstreamURIs=grpc://localhost:9091 upstreamWorkers=1 listenAddress=0.0.0.0:9090 service type=http
web_1            | 2019-10-10T20:03:35.460Z [INFO]  Adding handler for UI static files
jaeger_1         | {"level":"info","ts":1570737815.9990659,"caller":"flags/service.go:115","msg":"Mounting metrics handler on admin server","route":"/metrics"}
jaeger_1         | {"level":"info","ts":1570737815.999266,"caller":"flags/admin.go:108","msg":"Mounting health check on admin server","route":"/"}
```

## Consul
Once running the consul UI is accessible at [http://localhost:8500](http://localhost:8500), you can also access the Consul API at the same address.
Service registration and the application of central config is applied when starting the application. The config files
can be found at the following locations:
* Consul Config [consul_config/config.hcl](consul_config/config.hcl)
* Central Config to set service protocol [central_config/](central_config/)
* Service Config for registering services and configuring upstreams [service_config/](service_config/)

![](images/consul-ui.png)

## Interacting with the application
The main entry point for the system is accessible at [http://localhost:9090](http://localhost:9090), you can either 
interact with it using the UI [http://localhost:9090/ui](http://localhost:9090/ui).

![](images/fake-ui.png)

Or by directly curling the main service API:

```
consul-demo-tracing on  master [?] via 🐹 v1.13.1
➜ curl localhost:9090
{
  "name": "Ingress",
  "uri": "/",
  "type": "HTTP",
  "start_time": "2019-10-10T20:10:11.590226",
  "end_time": "2019-10-10T20:10:11.630163",
  "duration": "39.9377ms",
  "body": "Hello World",
  "upstream_calls": [
    {
      "name": "Web",
      "uri": "http://localhost:9091",
      "type": "HTTP",
      "start_time": "2019-10-10T20:10:11.605254",
      "end_time": "2019-10-10T20:10:11.627063",
      "duration": "21.932ms",
      "body": "Web response",
      "upstream_calls": [
        {
          "name": "API",
          "uri": "grpc://localhost:9091",
          "type": "gRPC",
          "start_time": "2019-10-10T20:10:11.616201",
          "end_time": "2019-10-10T20:10:11.617864",
          "duration": "1.6636ms",
          "body": "API response",
          "code": 0
        }
      ],
      "code": 200
    }
  ],
  "code": 200
}
```

The API service has been configured to return an error approximately 20% of all calls.

## Tracing
The 3 services and their associated Envoy proxies are configured to emit tracing information in the Zipkin format which 
is collected by Jaeger. The Jaeger UI can be accessed at the following URL: [http://localhost:16686/search](http://localhost:16686/search).

### Example trace in Jaeger

![](jaeger.png)

### Example trace with errors

![](jaeger-with-error.png)

## Stopping the demo
To cleanly remove all containers and networks, please use the following command:

```
consul-demo-tracing/jaeger on  master [?] via 🐹 v1.13.1 on 🐳 v19.03.2 took 17m 4s
➜ docker-compose down
Removing jaeger_ingress_envoy_1 ... done
Removing jaeger_web_proxy_1     ... done
Removing jaeger_api_proxy_1     ... done
Removing jaeger_api_1           ... done
Removing jaeger_jaeger_1        ... done
Removing jaeger_consul_1        ... done
Removing jaeger_ingress_1       ... done
Removing jaeger_web_1           ... done
Removing network jaeger_vpcbr
```
