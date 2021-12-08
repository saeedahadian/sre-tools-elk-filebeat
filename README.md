# Filebeat

We use filebeat to simply send files' contents to Elasticsearch without doing any sort of pre-processing on them.

## Input

To send log files, we use the filestream input. It's as easy as giving the address in which log files are located and filebeat will pull the log files and send them to the determined output channel. We can also exclude some lines before sending them to the output. For instance, here we exclude all error lines cause they are caught by APM.

```yaml
  exclude_lines: ['^ERR']
```

Note that the paths we set here are inside the container. We could use docker-compose volume mounts to put all of our log files inside this directory for the filebeat to find them.

```yaml
# filebeat config
  paths:
    - '/var/log/filebeat/**/*.log'
```

```yaml
# docker-compose config
  volumes:
    - ../kyc_challenge_pose/apm_agent/trace_logs:/var/log/filebeat/kyc_challenge_pose:ro
    - ../kyc_face_verification/apm_agent/trace_logs:/var/log/filebeat/kyc_face_verification:ro
```

## Output

There are 2 outputs defined in the configuration file (one is commented out).

+ **logstash output**: This is the main output that we use to send filebeat's data for further processing to the logstash instances. We have set up2 logstash instances and are using them with filebeat's builtin load-balancer.

+ **console output**: This is output which is by default commented out could be used for the purpose of debugging. The output will show the data that filebeat is sending inside the output console which is easy to read and debug immediately.

## Processors

It is not that we cannot process filebeat's data at all. Some manipulations could be done on the data while in filebeat before sending the data to the output channels. Here we add an additional field to each document which shows the name, the version and the environment of the service.
