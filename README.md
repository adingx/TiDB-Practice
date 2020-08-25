# Performance Testing on TiDB v4.0.0
* GOAL: performance testing via sysbench, go-ycsb, go-tpc ;

## Hardware Configure 
* One Aliyun Cloud ECS, cpu 8cores, mem 32GB;
* Storage - 200GB SSD ;
* OS - ubuntu 18.04 ;

# Setup Tidb Runtime
`tiup playground v4.0.0 --db 1 --pd 1 --kv 3 --monitor`

### dashboard access via nginx reverse proxy 
```s
sudo apt install nginx 

EDIT /etc/nginx/sites-available/default
server {
    listen 80;
    server_name tidb-dashboard ;
    location / {
        proxy_pass http://127.0.0.1:2379;
        proxy_redirect off;
    }
}
sudo system restart nginx
```

* Open the URL - http://External-IP/dashboard/
* username: root ; the initial password is empty ;

<img align="center" src="/images/02-tidb_dashboard.png">


## Benchmark SETUP 

### sysbench 
* [sysbench](https://github.com/akopytov/sysbench) is a scriptable multi-threaded benchmark tool based on LuaJIT.

* Sysbench SETUP 
```s 
curl -s https://packagecloud.io/install/repositories/akopytov/sysbench/script.deb.sh | sudo bash
sudo apt -y install sysbench mysql-client-core-5.7

mysql --host 127.0.0.1 --port 4000 -u root
> create database sbtest;
```

* EDIT sbt-config
```s
mysql-host=127.0.0.1
mysql-port=4000
mysql-user=root
mysql-password=
mysql-db=sbtest
time=600
threads=16
report-interval=10
db-driver=mysql
```

* stuffing test data
```s
sysbench --config-file=sbt-config oltp_point_select --tables=32 --table-size=10000 prepare
```

* running select test
```s
sysbench --config-file=sbt-config oltp_point_select --threads=16 --tables=32 --table-size=10000 run
```

* Test report is [here](/txt/L2_sbt_select.txt).

* running update-index-field test
```s
sysbench --config-file=sbt-config oltp_update_index --threads=16 --tables=32 --table-size=10000 run
```

* Test report is [here](/txt/L2_sbt_update_idx.txt).


### go-ycsb 
* go-ycsb SETUP 
```s
wget https://dl.google.com/go/go1.15.linux-amd64.tar.gz
tar -C /usr/local -xzf go1.15.linux-amd64.tar.gz 
export PATH=$PATH:/usr/local/go/bin

git clone https://github.com/pingcap/go-ycsb
cd go-ycsb
make
```

* load data
```s
./bin/go-ycsb load mysql -P workloads/workloada -p recordcount=100000 -p mysql.host=127.0.0.1 -p mysql.port=4000 --threads 32
```

* test
```s
./bin/go-ycsb run mysql -P workloads/workloada -p operationcount=1000000 -p mysql.host=127.0.0.1 -p mysql.port=4000 --threads 64
```

* Test report is [here](/txt/L2_ycsb_wla.txt).

### go-tpc 

* go-tpc SETUP 
```s
git clone https://github.com/pingcap/go-tpc.git
cd go-tpc
make build
```

* TPC-C 
```s
./bin/go-tpc tpcc -H 127.0.0.1 -P 4000 -D tpcc --warehouses 100 prepare
./bin/go-tpc tpcc -H 127.0.0.1 -P 4000 -D tpcc --warehouses 100 run
```

* Test report is [here](/txt/L2_TPC_C.txt).

* TPC-H 
```s
./bin/go-tpc tpch prepare -H 127.0.0.1 -P 4000 -D tpch --sf 2 --analyze
./bin/go-tpc tpch run -H 127.0.0.1 -P 4000 -D tpch --sf 2
```

* Test report is [here](/txt/L2_TPC_H.txt).

## Analysis 

TODO.

