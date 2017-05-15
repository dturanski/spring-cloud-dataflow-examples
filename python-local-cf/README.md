# Python local processor on Cloud Foundry 

Run a stream `time | time-transformer | log`  where `time-transformer` runs a local Python app to process messages.


# cf push Spring Cloud Stream apps 

NOTE: For instructions to run this demo on pcfdev, see the last section.

## Assemble the `time-transformer` app

In this step you will copy a simple Python app and the Spring Cloud Stream local Python processor into a directory. You should end up with:

```
app\
   python-local-processor-rabbit-1.2.1.BUILD-SNAPSHOT.jar
   time-delta.py
   requirements.txt
``` 

```
$cd python-local-cf
$git clone https://github.com/dturanski/python-apps.git
$mkdir app
$cp python-apps/test-stream-scripts/time-transformer/* app
$cd app
$wget https://raw.githubusercontent.com/dturanski/spring-cloud-stream-binaries/master/binaries/python-local-processor-rabbit-1.2.1.BUILD-SNAPSHOT.jar
```
## Download time and log

```
$ wget https://repo.spring.io/libs-snapshot/org/springframework/cloud/stream/app/time-source-rabbit/1.2.1.BUILD-SNAPSHOT/time-source-rabbit-1.2.1.BUILD-SNAPSHOT.jar
$ wget https://repo.spring.io/libs-snapshot/org/springframework/cloud/stream/app/log-sink-rabbit/1.2.1.BUILD-SNAPSHOT/log-sink-rabbit-1.2.1.BUILD-SNAPSHOT.jar
```

## Push the apps

Note that `time-transformer-manifest.yml` is configured to use [The Java Python buildpack](https://github.com/dturanski/java-python-buildpack).

```
$ cd python-local-cf
$ cf push -f time-transformer-manifest.yml
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



# Instructions for running the demo on pcfdev

Start pcfdev and create a `rabbit` service if necessary.

NOTE: You will likely see disk full errors attempting to push `time-transformer` to pcfdev. The jar file is ~ 71 MB with jython-standalone embedded. This demo does not require Jython wrappers so you can build apps/python-local-processor-rabbit without Jython:

Use https://github.com/dturanski/spring-cloud-stream-binaries/blob/master/binaries/python-local-processor-rabbit-no-jython-1.2.1.BUILD-SNAPSHOT.jar?raw=true 

Or build one:

```
$ git clone https://github.com/dturanski/spring-cloud-stream-app-starters-python.git
$ cd spring-cloud-stream-app-starters-python
$ ./mvnw clean install -PgenerateApps
$ cd apps/python-local-processor-rabbit
```

Edit the pom file:

```xml
<dependency>
    <groupId>org.springframework.cloud.stream.app</groupId>
     <artifactId>spring-cloud-starter-stream-processor-python-local</artifactId>
     <exclusions>
         <exclusion>
            <groupId>org.python</groupId>
            <artifactId>jython-standalone</artifactId>
         </exclusion>
     </exclusions> 
</dependency>

```

```
$ mvn clean package
```

Increase default memory

```
$ cf push -f log-sink-manifest.yml -m 750M
$ cf push -f time-source-manifest.yml -m 750M
```


