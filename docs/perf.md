# perf


## Proiling a running JAVA application using perf (Linux)

A user ran a Java program (gatk) and complained that it is running slower than expected. We need to find out where it spends its time, and hopefully remove the bottleneck. The goal is to hook into the running process based on its Pid, and get sampled call info so as to find where most time is spent on. 

Using [async-profiler](https://github.com/jvm-profiling-tools/async-profiler) and [Flamegraphs](http://www.brendangregg.com/flamegraphs.html)

Use `top` command to list processes in the system and identify the Pid of the Java process of interest. Also note down the name of the user who ran the process. 

```bash
sudo -u <username> ./profiler.sh -d 60 -f flamegraph.svg -s -o svg 13644
```

13644 is the PID

flamegraph.svg is the name of the output file

60 is the sampling frequency

and this yields

![flamegraph](https://aravindh.net/wiki/images/flamegraph.svg)
[Interactive version](https://aravindh.net/wiki/images/flamegraph.svg)

