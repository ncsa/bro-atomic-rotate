bro-atomic-rotate
=================

Installation:

    cp zz_rotate /usr/local/bro/share/broctl/scripts/postprocessors/
    cp bro-atomic-rotate /usr/local/bin
    cp bro-atomic-rotate.service /etc/systemd/system
    systemctl enable bro-atomic-rotate
    systemctl start bro-atomic-rotate

# Background

## How does bro rotate/archive logs?
    mv conn.log conn-yaddayadda.log
    gzip < conn-yaddayadda.log > /bro/logs/2018/10/10/conn.09:00:00-10:00:00.gz

## What happens if?
* Power loss
* Reboot
* OOM
* Something is trying to read .gz files as they are created

## Aside: Atomic operations

### Atomic
    mv (on same filesystem)

### Not Atomic
    cp
    mv (across filesystems)
    curl -o file.csv http://example.com/file.csv 
    gzip < in > out

## Solution?
### mv (across filesystem)
    cp src dst.tmp && mv dst.tmp dst && rm src
    (mv will do cp src dst && rm src)
### Compress
    gzip < src > dst.tmp && mv dst.tmp dst && rm src
### Download
    curl --fail -o file.csv.tmp && mv file.csv.tmp file.csv

## How does bro (really) rotate/archive logs?
    gzip < conn-yaddayadda.log > /bro/logs/2018/10/10/conn.09:00:00-10:00:00.gz &
    gzip < dns-yaddayadda.log  > /bro/logs/2018/10/10/dns.09:00:00-10:00:00.gz  &
    gzip < http-yaddayadda.log > /bro/logs/2018/10/10/http.09:00:00-10:00:00.gz &
    gzip < ssl-yaddayadda.log  > /bro/logs/2018/10/10/ssl.09:00:00-10:00:00.gz  &

Bro does not rotate files one at a time, instead it starts one background task for each log file concurrently.

## Serialize
### Bro side
    mv conn-yaddayadda.log /bro/logs/log_queue/conn… &
    mv http-yaddayadda.log /bro/logs/log_queue/http… &
### External tool
    loop
        for each file in /bro/logs/log_queue 
            Atomically rotate $file
        sleep

The end result is that each log file will be archived serially and atomically.
