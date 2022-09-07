---
title: Diagnostic Package
language: en
sidebar_label: Diagnostic Package
pagination_label: Diagnostic Package
toc_min_heading_level: 2
toc_max_heading_level: 6
pagination_prev: null
pagination_next: null
keywords:
    - diagnostic package
draft: false
last_update:
    date: 08/16/2022
---

Kylin users may face with many problems during usage, such as failure in creating a model or query errors. Kylin provides a cli tool to pack related information into a zip package to help system administrator to better analyze the root cause of these problems.
Diagnosis function including System Diagnosis、Job Diagnosis and Query Diagnosis.

### Generate Diagnostic Package using bash script

- Generate system diagnostic package
Execute `$KYLIN_HOME/bin/diag.sh [-startTime <START_TIMESTAMP> -endTime <END_TIMESTAMP>] [-destDir <DESTINATION_DIR>]` to generate the system diagnostic package. The default time range is last 1 day, START_TIMESTAMP and END_TIMESTAMP is in unix timestamp. You may get the current unix timestamp by run ```echo $((`date +%s`*1000))``` in Linux.

- Generate job diagnostic package
Execute `$KYLIN_HOME/bin/diag.sh -job <jobid> [-destDir <DESTINATION_DIR>]` to generate the job diagnostic package with replacing `<jobid>` with the job ID number.

- Generate query diagnostic package
  Execute `$KYLIN_HOME/bin/diag.sh -project <project> -query <queryid> [-destDir <DESTINATION_DIR>]` to generate the query diagnostic package with replacing `<queryid>` with the actual query ID number and `<project>` with the name of the project in which the actual query is made.
  
- Diagnostic package storage location
Diagnostic packages generated by scripts are stored under `$KYLIN_HOME/diag_dump/` by default, you can specify the location by configuring `-destDir` parameter.
   > Note: In order to avoid the diagnostic package occupying a lot of storage, please clean up the `$KYLIN_HOME/diag_dump/` directory in time.

- Skip metadata files
    If you want to exclude metadata files, please specify `-includeMeta false`.

- Skip audit log files
    If you want to exclude audit log files, please specify `-includeAuditLog false` or add configuration `kylin.diag.include-auditlog=false` in the `kylin.properties`

### Diagnostic Package Content

#### Full Diagnostic Package Content

- `/conf`: configuration information under the `$KYLIN_HOME/conf` directory.
- `/hadoop_conf`: configuration information under the `$KYLIN_HOME/hadoop_conf` directory.
- `/metadata`: metadata files.
- `/logs`: all the logs in the specified time range, 1 day is by default.
- `/spark_logs`: all Spark executor logs of query in the specified time range.
- `/sparder_history`：all Sparder logs of query in  the specified time range.
- `/system_metrics`: all system metrics in the specified time range. 
- `/audit_log`: all audit logs of metdata in the specified time range.
- `/job_tmp`: the optimization suggestions log.
- `/client`: operating system resources occupied information, Hadoop version and Kerberos information.
- `/bin`: program execute and manager binary files.
- `/monitor_metrics`: System monitoring statistics.
- `/write_hadoop_conf`：`$KYLIN_HOME/write_hadoop_conf`, Hadoop configuration of the build cluster. This directory will not be available when Read/Write separation deployment is not configured.
- file `catalog_info`: directory structure of install package.
- file `commit_SHA1`: git-commit version.
- file `hadoop_env`: hadoop environment information.
- file `info`: license，package and hostname.
- file `kylin_env`: Kylin version, operating system information, Java related information, git-commit information.
- file `time_used_info`: Time statistics of each file generated in the diagnostic package.

#### Job Diagnostic Package Content

- `/conf`: configuration information under the `$KYLIN_HOME/conf` directory.
- `/hadoop_conf`: configuration information under the `$KYLIN_HOME/hadoop_conf` directory.
- `/job_history`：Job execution history information mainly includes the execution information of yarn application.
- `/metadata`: metadata files.
- `/logs`: the logs generated during the execution of the job.
- `/spark_logs`: all spark executor logs generated during job execution.
- `/system_metrics`: the system metrics generated during the execution.
- `/audit_log`: the audit logs generated during the execution.
- `/job_tmp`: the temporary files, logs and optimization suggestions log of job.
- `/yarn_application_log`: specifies the logs of yarn application of job. 
- `/client`: operating system resources occupied information, Hadoop version and Kerberos information.
- `/bin`: program execute and manager binary files.
- `/monitor_metrics`: System monitoring statistics.
- `/write_hadoop_conf`：`$KYLIN_HOME/write_hadoop_conf`, Hadoop configuration of the build cluster. This directory will not be available when Read/Write separation deployment is not configured.
- file `catalog_info`: directory structure of install package.
- file `commit_SHA1`: git-commit version.
- file `hadoop_env`: hadoop environment information.
- file `info`: license, package and hostname.
- file `kylin_env`：kyligence Enterprise version, operating system information, Java related information, git-commit information.
- file `time_used_info`: Time statistics of each file generated in the diagnostic package.

#### Query Diagnostic Package Content

- `/conf`: configuration information under the `$KYLIN_HOME/conf` directory.
- `/hadoop_conf`: configuration information under the `$KYLIN_HOME/hadoop_conf` directory.
- `/metadata`：specify the metadata for all models under the project.
- `/logs`：`$KYLIN_HOME/logs/kylin.log`，specify the log of the query.
- `/spark_logs`：all Spark executor logs within the time range are included in the query diagnostic package.
- `/sparder_history`：all Sparder logs within the time range are included in the query diagnostic package.
- `/client`：operating system resource usage, Version information for Hadoop, and Kerberos information.
- file `catalog_info`: directory structure of install package.
- file `commit_SHA1`: git-commit version.
- file `hadoop_env`: hadoop environment information.
- file `info`: license, package and hostname.
- file `kylin_env`：kyligence Enterprise version, operating system information, Java related information, git-commit information.
- file `time_used_info`: Time statistics of each file generated in the diagnostic package.

### Multi-Node Diagnosis
At present, there is no API to know which nodes are in the cluster. You need to record the deployed nodes by yourself, and then go to each node to generate diagnostic package separately. The method of generating the diagnostic package is the same as the above.

### Diagnostic package desensitization

The diagnostic package desensitization function can hide sensitive information in the diagnostic package, such as user names, passwords, etc. While helping users solve problems, it can meet users' data security requirements.

You can enable the diagnostic package desensitization function through the following configuration item:

```properties
## The desensitization level of the diagnostic package. RAW stands for no desensitization, OBF stands for desensitization
kylin.diag.obf.level=OBF
```

After the configuration is enabled, the diagnostic package generated through the Web UI or through the terminal CLI tool will be desensitized. The system will desensitize all files starting with `kylin.properties` in the `KYLIN_HONE/conf` directory. All configuration items including `password`, `user`, `zookeeper.zk-auth`, ` The configuration items of source.jdbc.pass` will be desensitized. The desensitization method is to replace the value of the configuration item with `<hidden>`.

### Common Questions

**Q: Why is the system diagnostic package log content incomplete?**

A: The extraction of the log is a text-based match (based on the minute-level time string). If the content is found to be incomplete, it may be that the specified timestamp is not converted to the corresponding one when converted to a time string. You can try to modify the time range and re-generate the diagnostic package.

**Q: Why is the `system_metrics` directory missing content in diagnostic package?**

A: `system_metrics` contains system metrics, which are stored in InfluxDB. You need to specify an RPC port when using InfluxDB to back up the data, so please verify whether the configuration item `kylin.metrics.influx-rpc-service-bind-address` in the file `$KYLIN_HOME/conf/kylin.properties` is correct.

**Q: How to deal with Out of Memory (OOM) problem that happens during diagnostic package generating?**

A: Please check the value of `JAVA_VM_XMX` in `conf/setenv.sh`, which is recommended to be adjusted to more than 4 times the size of metadata. For example, if the size of metadata is 1G, please set the value to 4G or above.

**Q: Why is the file of the exported model optimization suggestion 0KB? **

A: If the model has no optimization suggestions, then the optimization suggestions generated by the corresponding model will be 0KB.