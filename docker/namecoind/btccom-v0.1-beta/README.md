Docker for Namecoind btccom-v0.1-beta
============================

* OS: `Ubuntu 14.04 LTS`
* Docker Image OS: `Ubuntu 16.04 LTS`
* Namecoind: `btccom-v0.1-beta`

## Install Docker

```
# Use 'curl -sSL https://get.daocloud.io/docker | sh' instead of this line
# when your server is in China.
wget -qO- https://get.docker.com/ | sh

service docker start
service docker status
```

## Build Docker Images

```
cd /work

git clone https://github.com/btccom/btcpool.git
cd btcpool/docker/namecoind/btccom-v0.1-beta

# If your server is in China, please check "Dockerfile" and uncomment some lines

# build
docker build -t namecoind:btccom-v0.1-beta .
# docker build --no-cache -t namecoind:btccom-v0.1-beta .

# mkdir for namecoind
mkdir -p /work/namecoind

# namecoin.conf
touch /work/namecoind/namecoin.conf
```

### namecoin.conf example

```
rpcuser=namecoinrpc
# generate random rpc password:
#   $ strings /dev/urandom | grep -o '[[:alnum:]]' | head -n 30 | tr -d '\n'; echo
rpcpassword=xxxxxxxxxxxxxxxxxxxxxxxxxx
rpcthreads=4

rpcallowip=172.16.0.0/12
rpcallowip=192.168.0.0/16
rpcallowip=10.0.0.0/8

# we use ZMQ to get notification when new block is coming
zmqpubrawblock=tcp://0.0.0.0:8337
zmqpubrawtx=tcp://0.0.0.0:8337
zmqpubhashtx=tcp://0.0.0.0:8337
zmqpubhashblock=tcp://0.0.0.0:8337

# use 1MB block when call GBT
blockmaxsize=1000000
```

## Start Docker Container

```
# start docker
docker run -it -v /work/namecoind:/root/.namecoin --name namecoind -p 8334:8334 -p 8336:8336 -p 8337:8337 --restart always -d namecoind:btccom-v0.1-beta

# login
docker exec -it namecoind /bin/bash
```

## Service Monitor

There are two cron jobs in docker by default. Editing or deleting it by your want:

```bash
$ docker exec -it namecoind /bin/bash
$ vim /etc/cron.d/namecoind
SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

* * * * *   root bash /root/scripts/opsgenie-monitor-namecoind.sh > /dev/null 2>&1
*/5 * * * * root bash /root/scripts/watch-namecoind.sh            > /dev/null 2>&1
```

### `opsgenie-monitor-namecoind.sh`

It provides a monitor service with heartbeats of opsgenie.

Setting `SERVICE` and `API_KEY` are necessary before it works.

```bash
$ vim /root/scripts/opsgenie-monitor-namecoind.sh
...

# the name of https://app.opsgenie.com/heartbeat
SERVICE="namecoind.01"

# api key of opsgenie
API_KEY="xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"

...
```

### `watch-namecoind.sh`
It calls `namecoin-cli getauxblock` and while the result is error/unexpected, it calls `namecoin-cli stop`. Then namecoind will be auto restarted by it's daemon.

By default, the script runs per 5 minutes. Maybe we should disable it at the block-synchronization stage of namecoind, or it will be restarted regularly per 5 minutes and gearing down the speed of block-downloading.


