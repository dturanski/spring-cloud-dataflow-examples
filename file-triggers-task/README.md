File Triggers Task Example
==

spring-cloud-task/spring-cloud-task-samples/taskprocessor

```
dataflow:>app register --type processor --name tlr-processor --uri maven://io.spring.cloud:taskprocessor:1.2.0.BUILD-SNAPSHOT
```

```
dataflow:>stream create file_trigger_task --definition "file --directory=/Users/dturanski/trigger_files --mode=ref |tlr-processor | task-launcher-local"
dataflow:>stream deploy file_trigger_task
```

will be in GA release
https://github.com/spring-cloud-task-app-starters/composed-task-runner/blob/master/spring-cloud-starter-task-composedtaskrunner/README.adoc
