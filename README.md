# How to run memtier on k8s

## Multple Options
The following are mutliple options of running memtier and testing latency. 
1. Run memtier from laptop. Enable port-forward to database service. 
2. Login to the least busy pod and run memtier. 
3. Run memtier workload in kubernetes cluster. This also allows ability to run paralle memtier workloads increasing load. 

## Required configurations

1. Set the proxy threads higher
   ```
	tune proxy all threads 8
	tune proxy all max_threads 16
   ```
   *note*: update all master shards proxy type or all nodes depending on scenario. 
   ```
   rladmin bind db <db_name> endpoint <endpoint id> policy <all-master-shards|all-nodes>
   ```
## Run a Workload in Kubernetes. 

<a href="workload"></a>
To test deployment and prove awesomeness of redis lets use  `memtier_benchmark` <a href="https://github.com/RedisLabs/memtier_benchmark" _target="blank">[link]</a>. Here are a couple of  deployment manifests: 

1. <a href="./benchmark.yml" _target="blank">Memtier without TLS</a>.
2. <a href="./benchmark-tls.yml" _target="blank">Memtier with TLS</a>, required when working through Openshift Routes.

Below is an example invocation of `memtier_benchmark` as from the commandline which is reflected in the manifest file linked above: <a href="./benchmark.yml" _target="blank">Memtier without TLS</a>. This invocation should yield somewhere between 3k and 10k requests per second. 

To generate more load we can adjust the `Limits` and `requests` values in this manifest: `memtier_benchmark` will consume as much resources as are given to it. 
We also also add replicas for parallel executions and multiply the load. 
```
memtier_benchmark -a 8ZSrzIK1 -s 172.30.94.228 -p 14000  "--pipeline=2", "--clients=4", "--threads=5", "--test-time=600"
```
What do the arguments above mean?
* `-a <password>`, `-s <server>`, `-p <port>`: default password  to connect to a Redis `server` on a specified `port`.
* `--tls --tls-skip-verify`: If we wish to Use TLS but to not verify server identity. 
* Data gen options 
  * Create `4` clients with `5` working threads each
  * Execute the test for 600secs  

To apply the benchmark workload: 
1. Update the arguments in the manifest to specific environment variable 
   * You can specify values directly with `env: {name, value}` pairs:
      ```
      env:
        - name: REDIS_PORT
          value: "14000"
      ```
   * These parameters can also be read from secrets in openshift.  
      ```
      - name: REDIS_PASSWORD
        valueFrom:
            secretKeyRef:
              key: password
              name: redb-redb-target
      ```
2. Apply the manifest: `oc apply -f benchmark.yaml`. If the arguments are properly specified then you will see a deployment and pod created for this workload. 

```
$ oc get all
  deployment.apps/redis-benchmark               2/2     2            2           5m44s
  pod/redis-benchmark-6675f4445-ckgk2              1/1     Running             0          5m44s
  pod/redis-benchmark-6675f4445-gj9sf              1/1     Running             0          5m44s
```

Alas, this is not a `memtier_benchmark` tutorial. Feel free to try out some of the other command line options. 
