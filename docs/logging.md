# Logging in Torchserve

In this document we explain logging in TorchServe. We also explain how to modify the behavior of logging in the model server.
Logging in TorchServe also covers metrics, as metrics are logged into a file.
To further understand how to customize metrics or define custom logging layouts, see [Metrics on TorchServe](metrics.md)

## Pre-requisites

* Be familiar with log4j configuration properties.
For information on how to configure log4j parameters, see [Logging Services](https://logging.apache.org/log4j/2.x/manual/configuration.html).
* Be familiar with the default [log4j.properties](https://github.com/pytorch/serve/blob/master/frontend/server/src/main/resources/log4j.properties) used by TorchServe.

## Types of logs

TorchServe currently provides the following types of logs

1. Access logs
1. TorchServe logs

### Access Logs

These logs collect the access pattern to TorchServe. The configuration for access logs are as follows:

```properties
log4j.logger.ACCESS_LOG = INFO, access_log


log4j.appender.access_log = org.apache.log4j.RollingFileAppender
log4j.appender.access_log.File = ${LOG_LOCATION}/access_log.log
log4j.appender.access_log.MaxFileSize = 100MB
log4j.appender.access_log.MaxBackupIndex = 5
log4j.appender.access_log.layout = org.apache.log4j.PatternLayout
log4j.appender.access_log.layout.ConversionPattern = %d{ISO8601} - %m%n
```

As defined in the properties file, the access logs are collected in {LOG_LOCATION}/access_log.log file.
When you load TorchServe with a model and run inference against the server, the following logs are collected into the access_log.log:

```text
2018-10-15 13:56:18,976 [INFO ] BackendWorker-9000 ACCESS_LOG - /127.0.0.1:64003 "POST /predictions/resnet-18 HTTP/1.1" 200 118
```

The above log tells us that a successful `POST` call to `/predictions/resnet-18` was made by remote host `127.0.0.1:64003` it took `118`ms to complete this request.

These logs are useful to determine the current performance of the model-server as well as understand the requests received by model-server.

### TorchServe Logs

These logs collect all the logs from TorchServe and from the backend workers (the custom model code).
The default configuration pertaining to TorchServe logs are as follows:

```properties
log4j.logger.com.amazonaws.ml.ts = DEBUG, ts_log


log4j.appender.ts_log = org.apache.log4j.RollingFileAppender
log4j.appender.ts_log.File = ${LOG_LOCATION}/ts_log.log
log4j.appender.ts_log.MaxFileSize = 100MB
log4j.appender.ts_log.MaxBackupIndex = 5
log4j.appender.ts_log.layout = org.apache.log4j.PatternLayout
log4j.appender.ts_log.layout.ConversionPattern = %d{ISO8601} [%-5p] %t %c - %m%n
```

This configuration by default dumps all the logs above `DEBUG` level.

## Generate custom logs

You might want to generate custom logs. This could be for debugging purposes or to log any errors.
To do this, print the required logs to `stdout/stderr`.
TorchServe captures the logs generated by the backend workers and logs it into the log file. Some examples of logs are as follows:

1. Messages printed to stderr:

```text
2018-10-14 16:46:51,656 [WARN ] W-9000-stderr org.pytorch.serve.wlm.WorkerLifeCycle - [16:46:51] src/nnvm/legacy_json_util.cc:209: Loading symbol saved by previous version v0.8.0. Attempting to upgrad\
e...
2018-10-14 16:46:51,657 [WARN ] W-9000-stderr org.pytorch.serve.wlm.WorkerLifeCycle - [16:46:51] src/nnvm/legacy_json_util.cc:217: Symbol successfully upgraded!
```

1. Messages printed to stdout:

```text
2018-10-14 16:59:59,926 [INFO ] W-9000-stdout org.pytorch.serve.wlm.WorkerLifeCycle - preprocess time: 3.60
2018-10-14 16:59:59,926 [INFO ] W-9000-stdout org.pytorch.serve.wlm.WorkerLifeCycle - inference time: 117.31
2018-10-14 16:59:59,926 [INFO ] W-9000-stdout org.pytorch.serve.wlm.WorkerLifeCycle - postprocess time: 8.52
```

## Modify the behavior of the logs

To modify the default logging behavior, define a `log4j.properties` file. There are two ways of starting TorchServe with custom logs:

### Provide with config.properties

 After you define a custom `log4j.properties` file, add the following to the `config.properties` file:

```properties
vmargs=-Dlog4j.configuration=file:///path/to/custom/log4j.properties
```

Then start TorchServe as follows:

```bash
$ torchserve --start --ts-config /path/to/config.properties
```

## Log with the TorchServe CLI

Alternatively, you could start the TorchServe with the following command as well

```bash
$ torchserve --start --log-config /path/to/custom/log4j.properties
```

## Enable asynchronous logging

If your model is super lightweight and you want high throughput, consider enabling asynchronous logging.
Log output might be delayed, and the most recent log might be lost if TorchServe is terminated unexpectedly.
Asynchronous logging is disabled by default.
To enable asynchronous logging, add following property in `config.properties`:

```properties
async_logging=true
```
