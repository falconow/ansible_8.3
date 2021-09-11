## Домашнее задание к занятию "08.03 Использование Yandex Cloud"
***
### Задание 1-4
> Дописал playbook на установку и настройку kibana и filebeat.

> Создал 3 виртуальных машины в Yandex Cloud, сформировал файл prod.yml
***
### Задание 5
> Проверям playbook на ошибки
```
falconow@falconow:~/lesson/ansible_8.3$ ansible-lint site.yml 
WARNING  Overriding detected file kind 'yaml' with 'playbook' for given positional argument: site.yml
WARNING  Listing 6 violation(s) that are fatal
risky-file-permissions: File permissions unset or incorrect
site.yml:9 Task/Handler: Upload .tar.gz file containing binaries from local storage

risky-file-permissions: File permissions unset or incorrect
site.yml:16 Task/Handler: Ensure installation dir exists

risky-file-permissions: File permissions unset or incorrect
site.yml:32 Task/Handler: Export environment variables

risky-file-permissions: File permissions unset or incorrect
site.yml:62 Task/Handler: Configure elasticsearch

risky-file-permissions: File permissions unset or incorrect
site.yml:94 Task/Handler: Configure Kibana

risky-file-permissions: File permissions unset or incorrect
site.yml:126 Task/Handler: Configure filebeat

You can skip specific rules or tags by adding them to your configuration file:
# .ansible-lint
warn_list:  # or 'skip_list' to silence them completely
  - experimental  # all rules tagged as experimental

Finished with 0 failure(s), 6 warning(s) on 1 files.
```
> Исправляем ошибки
```
falconow@falconow:~/lesson/ansible_8.3$ ansible-lint site.yml 
WARNING  Overriding detected file kind 'yaml' with 'playbook' for given positional argument: site.yml
falconow@falconow:~/lesson/ansible_8.3$ 
```
***
### Задание 6
```
falconow@falconow:~/lesson/ansible_8.3$ ansible-playbook -i inventory/prod.yml site.yml --check

PLAY [Install Java] *******************************************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************************************
ok: [application-instance]
ok: [el-instance]
ok: [k-instance]

TASK [Set facts for Java 11 vars] *****************************************************************************************************************************
ok: [el-instance]
ok: [k-instance]
ok: [application-instance]

TASK [Upload .tar.gz file containing binaries from local storage] *********************************************************************************************
changed: [application-instance]
changed: [el-instance]
changed: [k-instance]

TASK [Ensure installation dir exists] *************************************************************************************************************************
changed: [application-instance]
changed: [k-instance]
changed: [el-instance]

TASK [Extract java in the installation directory] *************************************************************************************************************
An exception occurred during task execution. To see the full traceback, use -vvv. The error was: NoneType: None
fatal: [application-instance]: FAILED! => {"changed": false, "msg": "dest '/opt/jdk/11.0.12' must be an existing dir"}
An exception occurred during task execution. To see the full traceback, use -vvv. The error was: NoneType: None
fatal: [k-instance]: FAILED! => {"changed": false, "msg": "dest '/opt/jdk/11.0.12' must be an existing dir"}
An exception occurred during task execution. To see the full traceback, use -vvv. The error was: NoneType: None
fatal: [el-instance]: FAILED! => {"changed": false, "msg": "dest '/opt/jdk/11.0.12' must be an existing dir"}

PLAY RECAP ****************************************************************************************************************************************************
application-instance       : ok=4    changed=2    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   
el-instance                : ok=4    changed=2    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   
k-instance                 : ok=4    changed=2    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   

falconow@falconow:~/lesson/ansible_8.3$ 
```
> Получаем ошибку, т.к. директория для распаковки Java не создана
***
 ### Задание 7
```
falconow@falconow:~/lesson/ansible_8.3$ ansible-playbook -i inventory/prod.yml site.yml --diff

PLAY [Install Java] *******************************************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************************************
ok: [application-instance]
ok: [el-instance]
ok: [k-instance]

TASK [Set facts for Java 11 vars] *****************************************************************************************************************************
ok: [el-instance]
ok: [k-instance]
ok: [application-instance]

TASK [Upload .tar.gz file containing binaries from local storage] *********************************************************************************************
diff skipped: source file size is greater than 104448
changed: [el-instance]
diff skipped: source file size is greater than 104448
changed: [application-instance]
diff skipped: source file size is greater than 104448
changed: [k-instance]

TASK [Ensure installation dir exists] *************************************************************************************************************************
--- before
+++ after
@@ -1,5 +1,5 @@
 {
-    "mode": "0755",
+    "mode": "0644",
     "path": "/opt/jdk/11.0.12",
-    "state": "absent"
+    "state": "directory"
 }

changed: [k-instance]
--- before
+++ after
@@ -1,5 +1,5 @@
 {
-    "mode": "0755",
+    "mode": "0644",
     "path": "/opt/jdk/11.0.12",
-    "state": "absent"
+    "state": "directory"
 }

changed: [application-instance]
--- before
+++ after
@@ -1,5 +1,5 @@
 {
-    "mode": "0755",
+    "mode": "0644",
     "path": "/opt/jdk/11.0.12",
-    "state": "absent"
+    "state": "directory"
 }

changed: [el-instance]

TASK [Extract java in the installation directory] *************************************************************************************************************
changed: [k-instance]
changed: [el-instance]
changed: [application-instance]

TASK [Export environment variables] ***************************************************************************************************************************
--- before
+++ after: /home/falconow/.ansible/tmp/ansible-local-19723vs_vssr8/tmpop9maf2w/jdk.sh.j2
@@ -0,0 +1,5 @@
+# Warning: This file is Ansible Managed, manual changes will be overwritten on next playbook run.
+#!/usr/bin/env bash
+
+export JAVA_HOME=/opt/jdk/11.0.12
+export PATH=$PATH:$JAVA_HOME/bin
\ No newline at end of file

changed: [el-instance]
--- before
+++ after: /home/falconow/.ansible/tmp/ansible-local-19723vs_vssr8/tmpgd2z2t57/jdk.sh.j2
@@ -0,0 +1,5 @@
+# Warning: This file is Ansible Managed, manual changes will be overwritten on next playbook run.
+#!/usr/bin/env bash
+
+export JAVA_HOME=/opt/jdk/11.0.12
+export PATH=$PATH:$JAVA_HOME/bin
\ No newline at end of file

changed: [k-instance]
--- before
+++ after: /home/falconow/.ansible/tmp/ansible-local-19723vs_vssr8/tmp3zeo7ro1/jdk.sh.j2
@@ -0,0 +1,5 @@
+# Warning: This file is Ansible Managed, manual changes will be overwritten on next playbook run.
+#!/usr/bin/env bash
+
+export JAVA_HOME=/opt/jdk/11.0.12
+export PATH=$PATH:$JAVA_HOME/bin
\ No newline at end of file

changed: [application-instance]

PLAY [Install elasticsearch] **********************************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************************************
ok: [el-instance]

TASK [Upload tar.gz Elasticsearch from remote URL] ************************************************************************************************************
changed: [el-instance]

TASK [Install rpm package elasticsearch] **********************************************************************************************************************
changed: [el-instance]

TASK [Configure elasticsearch] ********************************************************************************************************************************
--- before: /etc/elasticsearch/elasticsearch.yml
+++ after: /home/falconow/.ansible/tmp/ansible-local-19723vs_vssr8/tmpafrtk3rx/elasticsearch.yml.j2
@@ -14,13 +14,13 @@
 #
 # Use a descriptive name for your cluster:
 #
-#cluster.name: my-application
+cluster.name: my-application
 #
 # ------------------------------------ Node ------------------------------------
 #
 # Use a descriptive name for the node:
 #
-#node.name: node-1
+node.name: node-1
 #
 # Add custom attributes to the node:
 #
@@ -48,12 +48,13 @@
 #
 # Elasticsearch performs poorly when the system is swapping the memory.
 #
+#
 # ---------------------------------- Network -----------------------------------
 #
 # By default Elasticsearch is only accessible on localhost. Set a different
 # address here to expose this node on the network:
 #
-#network.host: 192.168.0.1
+network.host: 0.0.0.0
 #
 # By default Elasticsearch listens for HTTP traffic on the first free port it
 # finds starting at 9200. Set a specific HTTP port here:
@@ -68,10 +69,12 @@
 # The default list of hosts is ["127.0.0.1", "[::1]"]
 #
 #discovery.seed_hosts: ["host1", "host2"]
+discovery.seed_hosts: ["10.128.0.25"]
 #
 # Bootstrap the cluster using an initial set of master-eligible nodes:
 #
 #cluster.initial_master_nodes: ["node-1", "node-2"]
+cluster.initial_master_nodes: ["node-1"]
 #
 # For more information, consult the discovery and cluster formation module documentation.
 #

changed: [el-instance]

RUNNING HANDLER [Restart elascticsearch] **********************************************************************************************************************
changed: [el-instance]

PLAY [Install Kibana] *****************************************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************************************
ok: [k-instance]

TASK [Download Kibana rpm] ************************************************************************************************************************************
changed: [k-instance]

TASK [Install rpm package kibana] *****************************************************************************************************************************
changed: [k-instance]

TASK [Configure Kibana] ***************************************************************************************************************************************
--- before: /etc/kibana/kibana.yml
+++ after: /home/falconow/.ansible/tmp/ansible-local-19723vs_vssr8/tmp18e8y3tp/kibana.yml.j2
@@ -4,7 +4,7 @@
 # Specifies the address to which the Kibana server will bind. IP addresses and host names are both valid values.
 # The default is 'localhost', which usually means remote machines will not be able to connect.
 # To allow connections from remote users, set this parameter to a non-loopback address.
-#server.host: "localhost"
+server.host: "0.0.0.0"
 
 # Enables you to specify a path to mount Kibana at if you are running behind a proxy.
 # Use the `server.rewriteBasePath` setting to tell Kibana if it should remove the basePath
@@ -29,11 +29,11 @@
 #server.name: "your-hostname"
 
 # The URLs of the Elasticsearch instances to use for all your queries.
-#elasticsearch.hosts: ["http://localhost:9200"]
+elasticsearch.hosts: ["http://10.128.0.25:9200"]
 
 # Kibana uses an index in Elasticsearch to store saved searches, visualizations and
 # dashboards. Kibana creates a new index if the index doesn't already exist.
-#kibana.index: ".kibana"
+kibana.index: ".kibana"
 
 # The default application to load.
 #kibana.defaultAppId: "home"

changed: [k-instance]

RUNNING HANDLER [restart Kibana] ******************************************************************************************************************************
changed: [k-instance]

PLAY [Install filebeat] ***************************************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************************************
ok: [application-instance]

TASK [Download filebeat rpm] **********************************************************************************************************************************
changed: [application-instance]

TASK [Install rpm package filebeat] ***************************************************************************************************************************
changed: [application-instance]

TASK [Configure filebeat] *************************************************************************************************************************************
--- before: /etc/filebeat/filebeat.yml
+++ after: /home/falconow/.ansible/tmp/ansible-local-19723vs_vssr8/tmpcv3q7kwu/filebeat.yml.j2
@@ -1,270 +1,5 @@
-###################### Filebeat Configuration Example #########################
-
-# This file is an example configuration file highlighting only the most common
-# options. The filebeat.reference.yml file from the same directory contains all the
-# supported options with more comments. You can use it as a reference.
-#
-# You can find the full configuration reference here:
-# https://www.elastic.co/guide/en/beats/filebeat/index.html
-
-# For more available modules and options, please see the filebeat.reference.yml sample
-# configuration file.
-
-# ============================== Filebeat inputs ===============================
-
-filebeat.inputs:
-
-# Each - is an input. Most options can be set at the input level, so
-# you can use different inputs for various configurations.
-# Below are the input specific configurations.
-
-- type: log
-
-  # Change to true to enable this input configuration.
-  enabled: false
-
-  # Paths that should be crawled and fetched. Glob based paths.
-  paths:
-    - /var/log/*.log
-    #- c:\programdata\elasticsearch\logs\*
-
-  # Exclude lines. A list of regular expressions to match. It drops the lines that are
-  # matching any regular expression from the list.
-  #exclude_lines: ['^DBG']
-
-  # Include lines. A list of regular expressions to match. It exports the lines that are
-  # matching any regular expression from the list.
-  #include_lines: ['^ERR', '^WARN']
-
-  # Exclude files. A list of regular expressions to match. Filebeat drops the files that
-  # are matching any regular expression from the list. By default, no files are dropped.
-  #exclude_files: ['.gz$']
-
-  # Optional additional fields. These fields can be freely picked
-  # to add additional information to the crawled log files for filtering
-  #fields:
-  #  level: debug
-  #  review: 1
-
-  ### Multiline options
-
-  # Multiline can be used for log messages spanning multiple lines. This is common
-  # for Java Stack Traces or C-Line Continuation
-
-  # The regexp Pattern that has to be matched. The example pattern matches all lines starting with [
-  #multiline.pattern: ^\[
-
-  # Defines if the pattern set under pattern should be negated or not. Default is false.
-  #multiline.negate: false
-
-  # Match can be set to "after" or "before". It is used to define if lines should be append to a pattern
-  # that was (not) matched before or after or as long as a pattern is not matched based on negate.
-  # Note: After is the equivalent to previous and before is the equivalent to to next in Logstash
-  #multiline.match: after
-
-# filestream is an input for collecting log messages from files. It is going to replace log input in the future.
-- type: filestream
-
-  # Change to true to enable this input configuration.
-  enabled: false
-
-  # Paths that should be crawled and fetched. Glob based paths.
-  paths:
-    - /var/log/*.log
-    #- c:\programdata\elasticsearch\logs\*
-
-  # Exclude lines. A list of regular expressions to match. It drops the lines that are
-  # matching any regular expression from the list.
-  #exclude_lines: ['^DBG']
-
-  # Include lines. A list of regular expressions to match. It exports the lines that are
-  # matching any regular expression from the list.
-  #include_lines: ['^ERR', '^WARN']
-
-  # Exclude files. A list of regular expressions to match. Filebeat drops the files that
-  # are matching any regular expression from the list. By default, no files are dropped.
-  #prospector.scanner.exclude_files: ['.gz$']
-
-  # Optional additional fields. These fields can be freely picked
-  # to add additional information to the crawled log files for filtering
-  #fields:
-  #  level: debug
-  #  review: 1
-
-# ============================== Filebeat modules ==============================
-
-filebeat.config.modules:
-  # Glob pattern for configuration loading
-  path: ${path.config}/modules.d/*.yml
-
-  # Set to true to enable config reloading
-  reload.enabled: false
-
-  # Period on which files under path should be checked for changes
-  #reload.period: 10s
-
-# ======================= Elasticsearch template setting =======================
-
-setup.template.settings:
-  index.number_of_shards: 1
-  #index.codec: best_compression
-  #_source.enabled: false
-
-
-# ================================== General ===================================
-
-# The name of the shipper that publishes the network data. It can be used to group
-# all the transactions sent by a single shipper in the web interface.
-#name:
-
-# The tags of the shipper are included in their own field with each
-# transaction published.
-#tags: ["service-X", "web-tier"]
-
-# Optional fields that you can specify to add additional information to the
-# output.
-#fields:
-#  env: staging
-
-# ================================= Dashboards =================================
-# These settings control loading the sample dashboards to the Kibana index. Loading
-# the dashboards is disabled by default and can be enabled either by setting the
-# options here or by using the `setup` command.
-#setup.dashboards.enabled: false
-
-# The URL from where to download the dashboards archive. By default this URL
-# has a value which is computed based on the Beat name and version. For released
-# versions, this URL points to the dashboard archive on the artifacts.elastic.co
-# website.
-#setup.dashboards.url:
-
-# =================================== Kibana ===================================
-
-# Starting with Beats version 6.0.0, the dashboards are loaded via the Kibana API.
-# This requires a Kibana endpoint configuration.
+output.elasticsearch:
+  hosts: ["http://10.128.0.25:9200"]
 setup.kibana:
-
-  # Kibana Host
-  # Scheme and port can be left out and will be set to the default (http and 5601)
-  # In case you specify and additional path, the scheme is required: http://localhost:5601/path
-  # IPv6 addresses should always be defined as: https://[2001:db8::1]:5601
-  #host: "localhost:5601"
-
-  # Kibana Space ID
-  # ID of the Kibana Space into which the dashboards should be loaded. By default,
-  # the Default Space will be used.
-  #space.id:
-
-# =============================== Elastic Cloud ================================
-
-# These settings simplify using Filebeat with the Elastic Cloud (https://cloud.elastic.co/).
-
-# The cloud.id setting overwrites the `output.elasticsearch.hosts` and
-# `setup.kibana.host` options.
-# You can find the `cloud.id` in the Elastic Cloud web UI.
-#cloud.id:
-
-# The cloud.auth setting overwrites the `output.elasticsearch.username` and
-# `output.elasticsearch.password` settings. The format is `<user>:<pass>`.
-#cloud.auth:
-
-# ================================== Outputs ===================================
-
-# Configure what output to use when sending the data collected by the beat.
-
-# ---------------------------- Elasticsearch Output ----------------------------
-output.elasticsearch:
-  # Array of hosts to connect to.
-  hosts: ["localhost:9200"]
-
-  # Protocol - either `http` (default) or `https`.
-  #protocol: "https"
-
-  # Authentication credentials - either API key or username/password.
-  #api_key: "id:api_key"
-  #username: "elastic"
-  #password: "changeme"
-
-# ------------------------------ Logstash Output -------------------------------
-#output.logstash:
-  # The Logstash hosts
-  #hosts: ["localhost:5044"]
-
-  # Optional SSL. By default is off.
-  # List of root certificates for HTTPS server verifications
-  #ssl.certificate_authorities: ["/etc/pki/root/ca.pem"]
-
-  # Certificate for SSL client authentication
-  #ssl.certificate: "/etc/pki/client/cert.pem"
-
-  # Client Certificate Key
-  #ssl.key: "/etc/pki/client/cert.key"
-
-# ================================= Processors =================================
-processors:
-  - add_host_metadata:
-      when.not.contains.tags: forwarded
-  - add_cloud_metadata: ~
-  - add_docker_metadata: ~
-  - add_kubernetes_metadata: ~
-
-# ================================== Logging ===================================
-
-# Sets log level. The default log level is info.
-# Available log levels are: error, warning, info, debug
-#logging.level: debug
-
-# At debug level, you can selectively enable logging only for some components.
-# To enable all selectors use ["*"]. Examples of other selectors are "beat",
-# "publisher", "service".
-#logging.selectors: ["*"]
-
-# ============================= X-Pack Monitoring ==============================
-# Filebeat can export internal metrics to a central Elasticsearch monitoring
-# cluster.  This requires xpack monitoring to be enabled in Elasticsearch.  The
-# reporting is disabled by default.
-
-# Set to true to enable the monitoring reporter.
-#monitoring.enabled: false
-
-# Sets the UUID of the Elasticsearch cluster under which monitoring data for this
-# Filebeat instance will appear in the Stack Monitoring UI. If output.elasticsearch
-# is enabled, the UUID is derived from the Elasticsearch cluster referenced by output.elasticsearch.
-#monitoring.cluster_uuid:
-
-# Uncomment to send the metrics to Elasticsearch. Most settings from the
-# Elasticsearch output are accepted here as well.
-# Note that the settings should point to your Elasticsearch *monitoring* cluster.
-# Any setting that is not set is automatically inherited from the Elasticsearch
-# output configuration, so if you have the Elasticsearch output configured such
-# that it is pointing to your Elasticsearch monitoring cluster, you can simply
-# uncomment the following line.
-#monitoring.elasticsearch:
-
-# ============================== Instrumentation ===============================
-
-# Instrumentation support for the filebeat.
-#instrumentation:
-    # Set to true to enable instrumentation of filebeat.
-    #enabled: false
-
-    # Environment in which filebeat is running on (eg: staging, production, etc.)
-    #environment: ""
-
-    # APM Server hosts to report instrumentation results to.
-    #hosts:
-    #  - http://localhost:8200
-
-    # API Key for the APM Server(s).
-    # If api_key is set then secret_token will be ignored.
-    #api_key:
-
-    # Secret token for the APM Server(s).
-    #secret_token:
-
-
-# ================================= Migration ==================================
-
-# This allows to enable 6.7 migration aliases
-#migration.6_to_7.enabled: true
-
+  host: "http://10.128.0.19:5601"
+filebeat.config.modules.path: ${path.config}/modules.d/*.yml

changed: [application-instance]

TASK [Set filebeat systemwork] ********************************************************************************************************************************
changed: [application-instance]

TASK [Load Kibana dashboard] **********************************************************************************************************************************
ok: [application-instance]

RUNNING HANDLER [restart filebeat] ****************************************************************************************************************************
changed: [application-instance]

PLAY RECAP ****************************************************************************************************************************************************
application-instance       : ok=13   changed=9    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
el-instance                : ok=11   changed=8    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
k-instance                 : ok=11   changed=8    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

falconow@falconow:~/lesson/ansible_8.3$ 
```

> Проверям все ли запустилось:
```
falconow@falconow:~/lesson/ansible_8.3$ ssh 178.154.220.201
[falconow@el-instance ~]$ sudo systemctl status elasticsearch
● elasticsearch.service - Elasticsearch
   Loaded: loaded (/usr/lib/systemd/system/elasticsearch.service; enabled; vendor preset: disabled)
   Active: active (running) since Сб 2021-09-11 18:01:31 UTC; 9min ago
     Docs: https://www.elastic.co
 Main PID: 3002 (java)
   CGroup: /system.slice/elasticsearch.service
           ├─3002 /usr/share/elasticsearch/jdk/bin/java -Xshare:auto -Des.networkaddress.cache.ttl=60 -Des.networkaddress.cache.negative.ttl=10 -XX:+AlwaysP...
           └─3200 /usr/share/elasticsearch/modules/x-pack-ml/platform/linux-x86_64/bin/controller

сен 11 18:01:06 el-instance.ru-central1.internal systemd[1]: Starting Elasticsearch...
сен 11 18:01:31 el-instance.ru-central1.internal systemd[1]: Started Elasticsearch.
[falconow@el-instance ~]$ 
```
```falconow@falconow:~/lesson/ansible_8.3$ ssh 178.154.225.166
[falconow@k-instance ~]$ sudo systemctl status kibana
● kibana.service - Kibana
   Loaded: loaded (/etc/systemd/system/kibana.service; enabled; vendor preset: disabled)
   Active: active (running) since Сб 2021-09-11 18:03:39 UTC; 8min ago
     Docs: https://www.elastic.co
 Main PID: 3080 (node)
   CGroup: /system.slice/kibana.service
           ├─3080 /usr/share/kibana/bin/../node/bin/node /usr/share/kibana/bin/../src/cli/dist --logging.dest="/var/log/kibana/kibana.log" --pid.file="/run/...
           └─3102 /usr/share/kibana/node/bin/node --preserve-symlinks-main --preserve-symlinks /usr/share/kibana/src/cli/dist --logging.dest="/var/log/kiban...

сен 11 18:03:39 k-instance.ru-central1.internal systemd[1]: Started Kibana.
[falconow@k-instance ~]$ 
```
```
falconow@falconow:~/lesson/ansible_8.3$ ssh 62.84.114.50
[falconow@app-instance ~]$ sudo systemctl status filebeat
● filebeat.service - Filebeat sends log files to Logstash or directly to Elasticsearch.
   Loaded: loaded (/usr/lib/systemd/system/filebeat.service; enabled; vendor preset: disabled)
   Active: active (running) since Сб 2021-09-11 18:05:35 UTC; 7min ago
     Docs: https://www.elastic.co/beats/filebeat
 Main PID: 3088 (filebeat)
   CGroup: /system.slice/filebeat.service
           └─3088 /usr/share/filebeat/bin/filebeat --environment systemd -c /etc/filebeat/filebeat.yml --path.home /usr/share/filebeat --path.config /etc/fi...

сен 11 18:09:05 app-instance.ru-central1.internal filebeat[3088]: 2021-09-11T18:09:05.717Z        INFO        [monitoring]        log/log.go:145        N...
сен 11 18:09:35 app-instance.ru-central1.internal filebeat[3088]: 2021-09-11T18:09:35.717Z        INFO        [monitoring]        log/log.go:145        N...
сен 11 18:10:05 app-instance.ru-central1.internal filebeat[3088]: 2021-09-11T18:10:05.717Z        INFO        [monitoring]        log/log.go:145        N...
сен 11 18:10:35 app-instance.ru-central1.internal filebeat[3088]: 2021-09-11T18:10:35.717Z        INFO        [monitoring]        log/log.go:145        N...
сен 11 18:11:05 app-instance.ru-central1.internal filebeat[3088]: 2021-09-11T18:11:05.717Z        INFO        [monitoring]        log/log.go:145        N...
сен 11 18:11:35 app-instance.ru-central1.internal filebeat[3088]: 2021-09-11T18:11:35.717Z        INFO        [monitoring]        log/log.go:145        N...
сен 11 18:11:45 app-instance.ru-central1.internal filebeat[3088]: 2021-09-11T18:11:45.769Z        INFO        [input.harvester]        log/harvester.go:3...
сен 11 18:12:05 app-instance.ru-central1.internal filebeat[3088]: 2021-09-11T18:12:05.717Z        INFO        [monitoring]        log/log.go:145        N...
сен 11 18:12:35 app-instance.ru-central1.internal filebeat[3088]: 2021-09-11T18:12:35.721Z        INFO        [monitoring]        log/log.go:145        N...
сен 11 18:12:35 app-instance.ru-central1.internal filebeat[3088]: 2021-09-11T18:12:35.773Z        INFO        [input.harvester]        log/harvester.go:3...
Hint: Some lines were ellipsized, use -l to show in full.
[falconow@app-instance ~]$ 
```

> Все запустилось, проверил работу в браузере

***

### Задание 8
```
falconow@falconow:~/lesson/ansible_8.3$ ansible-playbook -i inventory/prod.yml site.yml --diff

PLAY [Install Java] *******************************************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************************************
ok: [application-instance]
ok: [k-instance]
ok: [el-instance]

TASK [Set facts for Java 11 vars] *****************************************************************************************************************************
ok: [el-instance]
ok: [k-instance]
ok: [application-instance]

TASK [Upload .tar.gz file containing binaries from local storage] *********************************************************************************************
ok: [application-instance]
ok: [k-instance]
ok: [el-instance]

TASK [Ensure installation dir exists] *************************************************************************************************************************
ok: [k-instance]
ok: [application-instance]
ok: [el-instance]

TASK [Extract java in the installation directory] *************************************************************************************************************
skipping: [el-instance]
skipping: [application-instance]
skipping: [k-instance]

TASK [Export environment variables] ***************************************************************************************************************************
ok: [el-instance]
ok: [application-instance]
ok: [k-instance]

PLAY [Install elasticsearch] **********************************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************************************
ok: [el-instance]

TASK [Upload tar.gz Elasticsearch from remote URL] ************************************************************************************************************
ok: [el-instance]

TASK [Install rpm package elasticsearch] **********************************************************************************************************************
ok: [el-instance]

TASK [Configure elasticsearch] ********************************************************************************************************************************
ok: [el-instance]

PLAY [Install Kibana] *****************************************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************************************
ok: [k-instance]

TASK [Download Kibana rpm] ************************************************************************************************************************************
ok: [k-instance]

TASK [Install rpm package kibana] *****************************************************************************************************************************
ok: [k-instance]

TASK [Configure Kibana] ***************************************************************************************************************************************
ok: [k-instance]

PLAY [Install filebeat] ***************************************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************************************
ok: [application-instance]

TASK [Download filebeat rpm] **********************************************************************************************************************************
ok: [application-instance]

TASK [Install rpm package filebeat] ***************************************************************************************************************************
ok: [application-instance]

TASK [Configure filebeat] *************************************************************************************************************************************
ok: [application-instance]

TASK [Set filebeat systemwork] ********************************************************************************************************************************
ok: [application-instance]

TASK [Load Kibana dashboard] **********************************************************************************************************************************
ok: [application-instance]

PLAY RECAP ****************************************************************************************************************************************************
application-instance       : ok=11   changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
el-instance                : ok=9    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
k-instance                 : ok=9    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   

falconow@falconow:~/lesson/ansible_8.3$ 
```

> playbook идемпотентен




