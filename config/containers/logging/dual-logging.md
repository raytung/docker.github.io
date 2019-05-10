---
description: Learn how to read container logs locally when using a third party logging solution.
keywords: docker, logging, driver
title: Reading container logs locally with remote logging drivers
---

## Overview 

In previous versions of Docker Enterprise, the jsonfile and journald log drivers supported reading 
container logs locally using `docker logs`. However, many third party logging drivers had no 
support for locally reading logs using `docker logs`, including: 

- syslog	
- gelf	
- fluentd	
- awslogs	
- splunk	
- etwlogs	
- gcplogs	
- Logentries

This created multiple problems, especially with UCP, when attempting to gather log data in an 
automated and standard way. Log information could only be accessed and viewed through the 
third-party solution in the format specified by that third-party tool, which often required authentication. 

Starting with Docker Engine Enterprise 18.03.1-ee-1, you can use `docker logs` to read container 
logs regardless of the configured logging solution or plugin. This capability, sometimes referred to 
as dual logging, allows you to use `docker logs` to read container logs locally in a consistent format, 
regardless of the remote log driver used, because the engine is configured to log information to the “local” 
logging driver. Refer to [Configure the default logging driver](/configure) for additional information. 

**Note**: 

- Dual logging is only available if you are using a third-party logging solution that does not 
support reading logs via 'docker logs'. 
- Dual logging is only supported for Docker Enterprise, and is enabled by default starting with 
Engine Enterprise 18.03.1-ee-1.

## Usage
Dual logging is enabled by default. You must configure either the docker daemon or the container with remote logging driver. 

The following example shows the results of running a `docker logs` command with and wihtout dual logging availability:

### Without dual logging capability:
When a container or `dockerd` was configured with a remote logging driver such as splunk, an error was 
displayed when attempting to read container logs locally:

- Step 1: Configure Docker daemon

    ```
    $ cat /etc/docker/daemon.json
    {
      "log-driver": "splunk",
      "log-opts": {
        ...
      }
    }
    ```

- Step 2: Start the container

    ```
    $ docker run -d busybox --name testlog top 
    ```

- Step 3: Read the container logs
    ```
    $ docker logs 7d6ac83a89a0
    ```
    The docker logs command was not available for drivers other than json-file and journald and an error was displayed.

### With dual logging capability:
To configure a container or docker with a remote logging driver such as splunk:

- Step 1: Configure Docker daemon
    ```
    $ cat /etc/docker/daemon.json
    {
      "log-driver": "splunk",
      "log-opts": {
        ...
      }
    }
    ```

- Step 2: Start the container
    ```
    $ docker run -d busybox --name testlog top 
    ```

- Step 3: Read the container logs
    ```
    $ docker logs 7d6ac83a89a0
    2019-02-04T19:48:15.423Z [INFO]  core: marked as sealed                                          	 
    2019-02-04T19:48:15.423Z [INFO]  core: pre-seal teardown starting                                                                                                 	 
    2019-02-04T19:48:15.423Z [INFO]  core: stopping cluster listeners                                                                                             	 
    2019-02-04T19:48:15.423Z [INFO]  core: shutting down forwarding rpc listeners                                                                                 	 
    2019-02-04T19:48:15.423Z [INFO]  core: forwarding rpc listeners stopped
    2019-02-04T19:48:15.599Z [INFO]  core: rpc listeners successfully shut down
    2019-02-04T19:48:15.599Z [INFO]  core: cluster listeners successfully shut down	
    ```

Note:
For a local driver, such as json-file and journald, there is no difference in functionality 
before or after the dual logging capability became available. The log is locally visible in both scenarios.


## Limitations

- You cannot specify more than one log driver. 
- If a container using a logging driver or plugin that sends logs remotely suddenly has a "network" issue, 
no ‘write’ to the local cache occurs. 
- If a write to `logdriver` fails for any reason (file system full, write permissions removed), 
the cache write fails and is logged in the daemon log. The log entry to the cache is not retried.
- Some logs might be lost from the cache in the default configuration because a ring buffer is used to 
prevent blocking the stdio of the container in case of slow file writes. An admin must repair these while the daemon is shut down.