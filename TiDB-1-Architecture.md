---
title: TiDB-1 Architecture
date: 2020-08-16 21:53:25
tags: tidb
---

## Introduction 
[TiDB](https://docs.pingcap.com/tidb/stable) is an open-source, distributed, NewSQL database that supports Hybrid Transactional and Analytical Processing (HTAP) workloads. It is MySQL compatible and features horizontal scalability, strong consistency, and high availability. TiDB can be deployed on-premise or in-cloud.

<img align="center" src="/images/TiDB-01-Arch.png">
<h3 style="text-align: center;">fig-1: TiDB Architecture</h3>

* TiDB - Database storage ;
* TiKV - key/value memory cache server ;
* PD(Placement Driver) - manage and schedule the TiKV cluster ;

## Deploy a TiDB Running Environment Using TiUP

[TiUP](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup) is a cluster operation and maintenance tool which provides TiUP cluster, a cluster management component written in Golang. 

## A Practice 
GOAL: Build TiDB/TiKV/PD from source code, and output a log "hello transaction." at transaction running time.

#### Setup Environment
TEST OS: ubuntu18.04 AMD x64 ;
* Install basic dependencies on ubuntu ;
`sudo apt install -y mysql-client-core-5.7 unzip cmake`

* Install rust ;
`curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh`

* Install golang ;
```s
wget https://dl.google.com/go/go1.15.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.15.linux-amd64.tar.gz 
export PATH=$PATH:/usr/local/go/bin
```

#### Building 
* Building TiKV ;
```s
git clone https://github.com/tikv/tikv.git
cd tikv
rustup component add rustfmt
rustup component add clippy
make build
```
the target binary is placed at `target/debug/tikv-server`.

* Building PD ;
```s
git clone https://github.com/pingcap/pd.git
cd pd
make
```
the target binary is placed at `bin/pd-server`.

* Building TiDB ;
```s
git clone https://github.com/pingcap/tidb
cd tidb
make
```
the target binary is placed at `bin/tidb-server`.

#### Hacking 
For transaction operation for tidb, in source code folder, txn.go is the related source file.
* Background transaction - `./kv/txn.go:RunInNewTxn` and `./store/tikv/txn.go:Commit`.
* SQL transaction - `./session/txn.go:Commit`.
* Insert the following line into these 3 function to check logging.
`logutil.Logger(ctx).Debug("store/tikv/txn.go Commit", zap.String("test", "hello transaction"))`


#### Running 
* Setup using TiUP
```s
curl --proto '=https' --tlsv1.2 -sSf https://tiup-mirrors.pingcap.com/install.sh | sh
source .bashrc
tiup playground v4.0.0 --db 1 --pd 1 --kv 3 --monitor
```

* Copy tikv/pd/tidb built targets to ~/.tiup/components/.. folder correspondingly .

* Checking the result from logging at ~/.tiup/data/RAMDON-NUM/tidb-0/tidb.log

<img align="center" src="/images/TiDB-01-result.png">
<h3 style="text-align: center;">fig-2: Running result.</h3>


## Reference
[pingCap-Tidb](https://docs.pingcap.com/tidb/stable)
[Tidb-github](https://github.com/pingcap/tidb)


