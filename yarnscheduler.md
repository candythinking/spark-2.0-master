## YarnScheduler - TaskScheduler for Client Deploy Mode {#__a_id_yarnscheduler_a_yarnscheduler_taskscheduler_for_client_deploy_mode}

YarnScheduler 是客户端部署模式下 YARN 上 Spark 的 TaskScheduler。

它是一个自定义 TaskSchedulerImpl，能够计算每个主机的机架，即它有一个专门的 getRackForHost。

如果尚未设置，它还会将 org.apache.hadoop.yarn.util.RackResolver logger 设置为 WARN。

### Tracking Racks per Hosts and Ports \(getRackForHost method\) {#__a_id_getrackforhost_a_tracking_racks_per_hosts_and_ports_getrackforhost_method}

getRackForHost 尝试计算主机的机架。

| Note | `getRackForHost`overrides the parent TaskSchedulerImpl’s getRackForHost |
| :---: | :--- |


它只是使用 Hadoop 的 org.apache.hadoop.yarn.util.RackResolver 来解析主机名到其网络位置，即机架。















