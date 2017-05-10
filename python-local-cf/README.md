# Python local processor on Cloud Foundry (PCF Dev)

Run a stream `time | python-local | log`  where `python-local` uses a local external Python app to process messages.


## Start PCF Dev
```
$ cf dev start
$ cf create-service p-rabbitmq standard rabbit
```

## Start CloudFoundry Dataflow Server

Note the following command line argument in conjunction with setting these environment variables, included in the environment settings as shown, enable the applications to be deployed with a 512MB JVM. This saves a lot of memory when running simple demos in pcfdev but YMMV:


`export SPRING_CLOUD_DEPLOYER_CLOUDFOUNDRY_STREAM_MEMORY=512`
`export SPRING_CLOUD_DEPLOYER_CLOUDFOUNDRY_STREAM_USE_SPRING_APPLICATION_JSON=false`


```
$ source dataflow_pcfdev_local.env
$ java -jar ./spring-cloud-dataflow-server-cloudfoundry-1.2.0.BUILD-SNAPSHOT.jar --spring.cloud.dataflow.applicationProperties.stream.JBP_CONFIG_OPEN_JDK_JRE='{ memory_calculator: { stack_threads: 1 } }'
```


## Start Shell
```
$ java -jar spring-cloud-dataflow-shell-1.2.0.BUILD-SNAPSHOT.jar
```

## Register apps
```
dataflow:>app import --uri http://bit.ly/Avogadro-SR1-stream-applications-rabbit-maven
dataflow:>app register --force --type processor --name python-local --uri https://raw.githubusercontent.com/dturanski/spring-cloud-stream-binaries/master/binaries/python-local-processor-rabbit-1.2.1.BUILD.jar

```

## Create the stream time | python-local | log

This clones the repo given by `git.uri`, installs dependent Python modules given by `python.basedir`/requirements.txt and runs the script given by
`python.basedir`/`python.script` in a separate process in the container. Message payloads are piped to/from the Python app via stdin/stdout. 

```
dataflow:>stream create pytest --definition "time | python-local --git.uri=https://github.com/dturanski/python-apps --python.basedir=test-stream-scripts/time-transformer --python.script=time-delta.py | log"

```
## Deploy the stream

```
dataflow:>stream deploy pytest --properties "deployer.python-local.cloudfoundry.buildpack=https://github.com/dturanski/java-python-buildpack"
```

## Tail the log output

```
$cf logs [random-prefix]-pytest-log
```

```
2017-05-09T23:06:33.56-0400 [APP/PROC/WEB/0]OUT 2017-05-10 03:06:33.565  INFO 6 --- [-local.pytest-1] log-sink                                 : 0:00:00.564542
2017-05-09T23:06:34.56-0400 [APP/PROC/WEB/0]OUT 2017-05-10 03:06:34.566  INFO 6 --- [-local.pytest-1] log-sink                                 : 0:00:00.564722
2017-05-09T23:06:35.57-0400 [APP/PROC/WEB/0]OUT 2017-05-10 03:06:35.569  INFO 6 --- [-local.pytest-1] log-sink                                 : 0:00:00.566075
2017-05-09T23:06:36.57-0400 [APP/PROC/WEB/0]OUT 2017-05-10 03:06:36.572  INFO 6 --- [-local.pytest-1] log-sink                                 : 0:00:00.571292
```
