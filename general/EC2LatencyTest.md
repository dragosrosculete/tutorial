## Latency test between two EC2 instances

Using netperf tool . Need to install on client and server
```
wget http://repo.iotti.biz/CentOS/7/x86_64/netperf-2.7.0-1.el7.lux.x86_64.rpmrpm -Uvh netperf-2.7.0-1.el7.lux.x86_64.rpmyum install netperf
rpm -Uvh netperf-2.7.0-1.el7.lux.x86_64.rpm
yum install netperf
```

On server run: \
```netserver -p 80```

On client run: \
```netperf -H <ip of server> -p 80 -t TCP_RR -l 50 -v 2 -- -O min_latency,mean_latency,max_latency,stddev_latency,transaction_rate```

Example output: 
```
Minimum         Mean         Maximum      Stddev       TransactionLatency
Latency         Latency      Latency      Latency      Rate
Microseconds    Microseconds Microseconds Microseconds Tran/s
87              112.49       500          13.07        8880.271
```