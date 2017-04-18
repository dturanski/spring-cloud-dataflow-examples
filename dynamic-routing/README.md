SCDF Dynamic Routing Example:
========


Download Local Server and Shell
----

```
$ wget http://repo.spring.io/milestone/org/springframework/cloud/spring-cloud-dataflow-server-local/1.2.0.M3/spring-cloud-dataflow-server-local-1.2.0.M3.jar
$ wget http://repo.spring.io/milestone/org/springframework/cloud/spring-cloud-dataflow-shell/1.2.0.M3/spring-cloud-dataflow-shell-1.2.0.M3.jar
```
Start Server and Shell
---

```
$java -jar spring-cloud-dataflow-server-local-1.2.0.M3.jar
$java -jar spring-cloud-dataflow-shell-1.2.0.M3.jar
```

Import OOTB Apps
---
```
dataflow:>app import http://bit.ly/Bacon-RELEASE-stream-applications-rabbit-maven
```

Create and Deploy Streams
---
```
dataflow:>stream create decision --definition "http --port=9000 | router --expression=#jsonPath(payload,'$.foo')" --deploy
dataflow:>stream create fooflow --definition ":foo > log" --deploy
dataflow:>stream create barflow --definition ":bar > log" --deploy
```

Tail Stdout Log Files
---

Find the name of the log files in the SCDF server console log. and run the tail command, e.g.,

```
$tail -f /var/folders/hd/5yqz2v2d3sxd3n879f4sg4gr0000gn/T/spring-cloud-dataflow-3948087009505024196/barflow-1492533260593/fooflow.log/stdout_0.log
$tail -f /var/folders/hd/5yqz2v2d3sxd3n879f4sg4gr0000gn/T/spring-cloud-dataflow-3948087009505024196/barflow-1492533260593/barflow.log/stdout_0.log
```


Post Messages
---
In a separate terminal window,

```
$curl -X POST -H "Content-Type:application/json" http://localhost:9000 -d '{"foo":"foo"}'
$curl -X POST -H "Content-Type:application/json" http://localhost:9000 -d '{"foo":"bar"}'
```
