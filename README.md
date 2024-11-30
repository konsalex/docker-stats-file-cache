Supportive material for my [post on my blog](https://blog.alexoglou.com/posts/docker-stats/)

---

To create artificial workload that will stress the IO of the database, we will use the `resourcestresser` benchmark from BenchBase.

## 1. Start the monitoring stack

```bash
docker-compose up -d
```

This will create the following services:
1. PostgreSQL
2. Prometheus
3. Grafana
4. Node Exporter
5. cAdvisor


## 2. Start loading data with BenchBase

```bash
docker run -it --env BENCHBASE_PROFILE='postgres' --network=host -v "$(pwd)/benchbase/resource_stresser_config.xml:/configs/resource_stresser_config.xml" benchbase.azurecr.io/benchbase -b resourcestresser -c "/configs/resource_stresser_config.xml" --create=true --load=true
```

When this step is completed, you should have around 20GB of data loaded in the PostgreSQL database.

To validate the data are there you can login to the database and run the following queries and you will get results looking along the lines of:

```
benchbase=# SELECT pg_size_pretty( pg_total_relation_size('iotablesmallrow'));
 pg_size_pretty
----------------
 200 MB
(1 row)

benchbase=# SELECT pg_size_pretty( pg_total_relation_size('iotable'));
 pg_size_pretty
----------------
 20 GB
(1 row)
```

## 3. Start executing the benchmark

The following command will start the benchmark, and then the IO will start getting under stress.

```bash
docker run -it --env BENCHBASE_PROFILE='postgres' --network=host -v "$(pwd)/benchbase/resource_stresser_config.xml:/configs/resource_stresser_config.xml" benchbase.azurecr.io/benchbase -b resourcestresser -c "/configs/resource_stresser_config.xml" --execute=true --create=false --load=false
```

## 4. Monitor the system

You can monitor the system by visiting the Grafana dashboard at [http://localhost:9100](http://localhost:9100).

For the rest of the analysis and the results, please refer to the [blog post](https://blog.alexoglou.com/posts/docker-stats/).