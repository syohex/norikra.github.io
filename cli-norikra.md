---
layout: layout
title: Norikra CLI
subtitle: Command line options of norikra
---
# Norikra CLI options

```
Usage:
  norikra start [-Xxxx] [other options]
```

```
Options:
  -H, [--host=HOST]                                      # host address that server listen [0.0.0.0]
  -P, [--port=N]                                         # port that server uses [26571]
      [--ui-port=N]                                      # port that Web UI server uses [26578]
  -s, [--stats=STATS]                                    # status file path to load/dump targets and queries [none]
      [--stats-secondary=STATS-SECONDARY]                # status file secondary path, to dump stats with date/time, like "stats.%Y%m%d.json" [none]
      [--suppress-dump-stat], [--no-suppress-dump-stat]  # specify not to update stat file with updated targets/queries on runtime [false]
      [--dump-stat-interval=N]                           # interval(seconds) of status file dumps on runtime [none (on shutdown only)]
      [--shutoff], [--no-shutoff]                        # [EXPERIMENTAL] enable "shutoff" mode, to reject input data under high memory usage
      [--shutoff-threshold=N]                            # threshold percent of heap memory usage to turn "shutoff mode" on
                                                         # Default: 90
      [--shutoff-check-interval=N]                       # interval seconds to turn "shutoff mode" on/off
                                                         # Default: 10
      [--ui-context-path=UI-CONTEXT-PATH]                # Web UI context path
  -d, [--daemonize], [--no-daemonize]                    # daemonize Norikra server [false (foreground)]
  -p, [--pidfile=PIDFILE]                                # pidfile path when daemonized [/var/run/norikra/norikra.pid]
                                                         # Default: /var/run/norikra/norikra.pid
      [--outfile=OUTFILE]                                # stdout redirect file when daemonized [${logdir}/norikra.out]
      [--bare-jvm], [--no-bare-jvm]                      # use JVM without any recommended options
      [--gc-log=GC-LOG]                                  # output gc logs on specified file path
      [--micro], [--no-micro]                            # development or testing (inbound:0, outbound:0, route:0, timer:0, rpc:2, web:2)
      [--small], [--no-small]                            # virtual or small scale servers (inbound:2, outbount:2, route:2, timer:2, rpc:9, web:9)
      [--middle], [--no-middle]                          # rackmount servers (inbound:4, outbound:4, route:4, timer:4, rpc:17, web:17)
      [--large], [--no-large]                            # high performance servers (inbound: 8, outbound: 8, route:8, timer:8, rpc:49, web:49)
      [--inbound-threads=N]                              # number of threads for inbound data
      [--outbound-threads=N]                             # number of threads for outbound data
      [--route-threads=N]                                # number of threads for events routing for query execution
      [--timer-threads=N]                                # number of threads for internal timers for query execution
      [--inbound-thread-capacity=N]                      
      [--outbound-thread-capacity=N]                     
      [--route-thread-capacity=N]                        
      [--timer-thread-capacity=N]                        
      [--rpc-threads=N]                                  # number of threads for rpc handlers
      [--web-threads=N]                                  # number of threads for WebUI handlers
  -l, [--logdir=LOGDIR]                                  # directory path of logfiles when daemonized [nil (console for foreground)]
      [--log-filesize=LOG-FILESIZE]                      # log rotation size [10MB]
      [--log-backups=N]                                  # log rotation backups [10]
      [--log-buffer-lines=N]                             # log lines to fetch from API [1000]
      [--log4j-properties-path=LOG4J-PROPERTIES-PATH]    # path to log4j.properties. ignore other log* options when this option is present
      [--more-quiet], [--no-more-quiet]                  # set loglevel as ERROR
  -q, [--quiet], [--no-quiet]                            # set loglevel as WARN
  -v, [--verbose], [--no-verbose]                        # set loglevel as DEBUG
      [--more-verbose], [--no-more-verbose]              # set loglevel as TRACE
  -h, [--help], [--no-help]                              # show this message
```

```
Usage:
  norikra stop [options]
  
Options:
  -p, [--pidfile=PIDFILE]  # pidfile path when daemonized [/var/run/norikra/norikra.pid]
                           # Default: /var/run/norikra/norikra.pid
      [--timeout=N]        # timeout seconds to wait process exit [5]
                           # Default: 5
```

## <a name="performance"></a>Performance option details

Threads option available with `norikra start`. Simple specifiers for performance with threadings:

    norikra start --micro     # or --small, --middle, --large (default: 'micro')

Norikra server has 3 types of threads:

* engine: 4 query engine thread types on Esper
  * inbound: input data handler for queries
  * outbound: output data handler for queries
  * router: event handler which decides which query needs that events
  * timer: executer for queries with time_batches and other timing events
* rpc: data input/output rpc handler threads on Jetty
* web: web ui request handler threads on Jetty

In many cases, norikra server handling high rate events needs large number of rpc threads to handle input/output rpc requests. WebUI don't need threads rather than default in almost all of cases.

Engine threads depends on queries running on norikra, input/output event data rate and target numbers. For more details, see Esper's API Documents: http://www.espertech.com/esper/release-5.2.0/esper-reference/html/api.html#api-threading

Norikra's simple specifiers details of threadings are:

* micro: development and testing
  * engine: all processings on single threads
  * rpc: 2 threads
  * web: 2 threads
* small: low rate events on virtual servers
  * engine: inbound 2, outbound 2, route 2, timer 2 threads
  * rpc: 9 threads
  * web: 9 threads
* middle: high rate events on physical servers
  * engine: inbound 4, outbound 4, route 4, timer 4 threads
  * rpc: 17 threads
  * web: 17 threads
* large: inbound heavy traffic and huge amount of queries
  * engine: inbound 8, outbound 8, route 8, timer 8 threads
  * rpc: 49 threads
  * web: 49 threads

To specify sizes of each threads, use `--*-threads=NUM` options.

### Jetty threads

Jetty requires threads more than CPU cores. If you execute Norikra on 4 CPU cores server, specify `--rpc-threads` and `--web-threads` more than 4.

## <a name="jvm_options"></a>JVM Options

`norikra start` commands passes following `-X` options into its JVM.

And, in default, `norikra start` specify jvm options below:

    -XX:-UseGCOverheadLimit
    -XX:+UseConcMarkSweepGC -XX:+UseCompressedOops
    -XX:CMSInitiatingOccupancyFraction=70 -XX:+UseCMSInitiatingOccupancyOnly
    -XX:NewRatio=1
    -XX:SurvivorRatio=2 -XX:MaxTenuringThreshold=15 -XX:TargetSurvivorRatio=80
    -XX:SoftRefLRUPolicyMSPerMB=200

Use `--bare-jvm` option if you want to specify these (or other conflicting) options.

For GC logs in trouble shootings, specifying `--gc-log FILE_PATH` make these options below enabled:

    -Xloggc:FILE_PATH
    -verbose:gc -XX:+PrintGCDetails -XX:+PrintGCDateStamps
    -XX:+HeapDumpOnOutOfMemoryError
