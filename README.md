# How to run memtier on k8s

1. Set the proxy threads higher
   ```
	tune proxy all threads 8
	tune proxy all max_threads 16
   ```
   *note*: ensure all master shards proxy type for the primary DB
   ```
   rladmin bind db <db_name> endpoint <endpoint id> policy <all-master-shards|all-nodes>
   ```
