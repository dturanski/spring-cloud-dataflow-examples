# Python local processor on Cloud Foundry 

Run a stream `time | python-local | log`  where `python-local` uses a local external Python app to process messages.


# cf push Spring Cloud Stream apps 

NOTE: Get disk full errors attempting to deploy to PCF Dev (The jar file is ~ 71 MB)


## Set up and run the Python processor

```
$cd python-local-cf
$git clone https://github.com/dturanski/python-apps.git
$mkdir app
$cp python-apps/test-stream-scripts/time-transformer/* app
$cd app
$wget https://raw.githubusercontent.com/dturanski/spring-cloud-stream-binaries/master/binaries/python-local-processor-rabbit-1.2.1.BUILD-SNAPSHOT.jar
$cd ..
$cf push -f time-transformer-manifest.yml
```

## Deploy  time and log

```
$ cd python-local-cf
$ wget https://repo.spring.io/libs-snapshot/org/springframework/cloud/stream/app/time-source-rabbit/1.2.1.BUILD-SNAPSHOT/time-source-rabbit-1.2.1.BUILD-SNAPSHOT.jar
$ wget https://repo.spring.io/libs-snapshot/org/springframework/cloud/stream/app/log-sink-rabbit/1.2.1.BUILD-SNAPSHOT/log-sink-rabbit-1.2.1.BUILD-SNAPSHOT.jar
$ cf push -f log-sink-manifest.yml
$ cf push -f time-source-manifest.yml
```

## Tail the log sink

```
$cf logs log

INFO 15 --- [2usR_jp7eqfvg-1] log                                      : 0:00:00.352073
INFO 15 --- [2usR_jp7eqfvg-1] log                                      : 0:00:00.352927
INFO 15 --- [2usR_jp7eqfvg-1] log                                      : 0:00:00.353800
INFO 15 --- [2usR_jp7eqfvg-1] log                                      : 0:00:00.355185
```

# Create the same stream with Dataflow

## Start CloudFoundry Dataflow Server

Configured for rabbit.


## Start Shell (or from GUI)
```
$ java -jar spring-cloud-dataflow-shell-1.2.0.BUILD-SNAPSHOT.jar

```

## Zip the python app
```
$ cd app
$ zip ../time-transformer.zip -r *

```

## Push the zip to a git repo or use the one in the example.


## Register apps

```
dataflow:>app import --uri http://bit.ly/Bacon-RELEASE-stream-applications
dataflow:>app register --type processor --name time-transformer --uri https://github.com/dturanski/spring-cloud-stream-binaries/blob/master/binaries/time-transformer.zip?raw=true
```


```

## Create and deploy the stream `time | time-transformer | log`
```
dataflow:>stream create pytock --definition "time | time-transformer --python.basedir=. --python.script=time-delta.py | log"
dataflow:>stream deploy pytock --properties "deployer.time-transformer.cloudfoundry.buildpack=https://github.com/dturanski/java-python-buildpack"
```

## Tail the log output

```
$cf logs [random-prefix]-pytock-log

INFO 15 --- [2usR_jp7eqfvg-1] log                                      : 0:00:00.352073
INFO 15 --- [2usR_jp7eqfvg-1] log                                      : 0:00:00.352927
INFO 15 --- [2usR_jp7eqfvg-1] log                                      : 0:00:00.353800
INFO 15 --- [2usR_jp7eqfvg-1] log                                      : 0:00:00.355185
```
