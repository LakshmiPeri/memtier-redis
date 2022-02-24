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
It's time to test your deployment. You can use the redis benchmarking tool `memtier_benchmark` <a href="https://github.com/RedisLabs/memtier_benchmark" _target="blank">[link]</a>. Here are a couple of examples deployment manifests: 

1. <a href="./benchmark.yml" _target="blank">Memtier without TLS</a>.
2. <a href="./benchmark-tls.yml" _target="blank">Memtier with TLS</a>, required when working through Openshift Routes.

Below is an example invocation of `memtier_benchmark` as from the commandline which is as-is reflected in the manifest file linked above: <a href="./benchmark-tls.yml" _target="blank">Benchmark with TLS</a>. This invocation should yield somewhere between 3k and 10k requests per second. If you want to generate more workload, adjust the `Limits` and `requests` values in this manifest: `memtier_benchmark` will consume as much resources as are given to it. 
```
memtier_benchmark -a 8ZSrzIK1 -s 172.30.94.228 -p 14000  --requests=2000000 --pipeline=1 --clients=4 --threads=5 --run-count=3
```
What do the arguments above mean?
* `-a <password>`, `-s <server>`, `-p <port>`: use Redis basic [`Auth`](https://redis.io/commands/auth) (without a specified user) to connect to a Redis `server` on a specified `port`.
* `--tls --tls-skip-verify`: If we wish to Use TLS but to not verify server identity. If you've installed your own server certs or installed our CA then `--tls-skip-verify` is likely unnecessary.
* Other options are fairly straight forward: 
  * Execute `2M` commands with only one command per requests (`--pipeline=1`)
  * Create `4` clients with `5` working threads each
  * Do all the above 3 times.  

To apply the benchmark workload: 
1. Edit the arguments in the file. 
   * You can specify values directly with `env: {name, value}` pairs:
      ```
      env:
        - name: REDIS_PORT
          value: "443"
      ```
   * You can also get the values from K8s Secrets as in the following: 
      ```
      - name: REDIS_PASSWORD
        valueFrom:
            secretKeyRef:
              key: password
              name: redb-redis-db1 
      ```
2. Apply the manifest: `oc apply -f benchmark.yaml`. If the arguments are properly specified then you will see a deployment and pod created for this workload. 

```
$ oc get all
  deployment.apps/redis-benchmark               2/2     2            2           5m44s
  pod/redis-benchmark-6675f4445-ckgk2              1/1     Running             0          5m44s
  pod/redis-benchmark-6675f4445-gj9sf              1/1     Running             0          5m44s
```

Alas, this is not a `memtier_benchmark` tutorial. Feel free to try out some of the other command line options. 
