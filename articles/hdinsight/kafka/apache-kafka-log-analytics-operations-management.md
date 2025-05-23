---
title: Azure Monitor logs for Apache Kafka - Azure HDInsight 
description: Learn how to use Azure Monitor logs to analyze logs from Apache Kafka cluster on Azure HDInsight.
ms.service: azure-hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
author: yeturis
ms.author: sairamyeturi
ms.reviewer: nijelsf
ms.date: 06/15/2024
---

# Analyze logs for Apache Kafka on HDInsight

Learn how to use Azure Monitor logs to analyze logs generated by Apache Kafka on HDInsight.

[!INCLUDE [azure-monitor-log-analytics-rebrand](~/reusable-content/ce-skilling/azure/includes/azure-monitor-log-analytics-rebrand.md)]

## Logs location

Apache Kafka logs in the cluster are located at `/var/log/kafka`. Kafka logs aren't saved or persisted across cluster life cycles, regardless if managed disks are used. The following table shows the available logs.

|Log |Description |
|---|---|
|kafka.out|stdout and stderr of the Kafka process. You'll find Kafka startup and shutdown logs in this file.|
|server.log|The main Kafka server log. All Kafka broker logs end up here.|
|controller.log|Controller logs if the broker is acting as controller.|
|statechange.log|All state change events to brokers are logged in this file.|
|kafka-gc.log|Kafka Garbage Collection stats.|

## Enable Azure Monitor logs for Apache Kafka

The steps to enable Azure Monitor logs for HDInsight are the same for all HDInsight clusters. Use the following links to understand how to create and configure the required services:

1. Create a Log Analytics workspace. For more information, see the [Logs in Azure Monitor](/azure/azure-monitor/logs/data-platform-logs) document.

2. Create a Kafka on HDInsight cluster. For more information, see the [Start with Apache Kafka on HDInsight](apache-kafka-get-started.md) document.

3. Configure the Kafka cluster to use Azure Monitor logs. For more information, see the [Use Azure Monitor logs to monitor HDInsight](../hdinsight-hadoop-oms-log-analytics-tutorial.md) document.

> [!IMPORTANT]  
> It may take around 20 minutes before data is available for Azure Monitor logs.

## Query logs

1. From the [Azure portal](https://portal.azure.com), select your Log Analytics workspace.

2. From the left menu, under **General**, select **Logs**. From here, you can search the data collected from Kafka. Enter a query in the query window and then select **Run**. The following are some example searches:

* Disk usage:

    ```kusto
    Perf
    | where ObjectName == "Logical Disk" and CounterName == "Free Megabytes" and InstanceName == "_Total" and ((Computer startswith_cs "hn" and Computer contains_cs "-") or (Computer startswith_cs "wn" and Computer contains_cs "-")) 
    | summarize AggregatedValue = avg(CounterValue) by Computer, bin(TimeGenerated, 1h)
    ```

* CPU usage:

    ```kusto
    Perf 
    | where CounterName == "% Processor Time" and InstanceName == "_Total" and ((Computer startswith_cs "hn" and Computer contains_cs "-") or (Computer startswith_cs "wn" and Computer contains_cs "-")) 
    | summarize AggregatedValue = avg(CounterValue) by Computer, bin(TimeGenerated, 1h)
    ```

* Incoming messages per second: (Replace `your_kafka_cluster_name` with your cluster name.)

    ```kusto
    metrics_kafka_CL 
    | where ClusterName_s == "your_kafka_cluster_name" and InstanceName_s == "kafka-BrokerTopicMetrics-MessagesInPerSec-Count" 
    | summarize AggregatedValue = avg(kafka_BrokerTopicMetrics_MessagesInPerSec_Count_value_d) by HostName_s, bin(TimeGenerated, 1h)
    ```

* Incoming bytes per second: (Replace `wn0-kafka` with a worker node host name.)

    ```kusto
    metrics_kafka_CL 
    | where HostName_s == "wn0-kafka" and InstanceName_s == "kafka-BrokerTopicMetrics-BytesInPerSec-Count" 
    | summarize AggregatedValue = avg(kafka_BrokerTopicMetrics_BytesInPerSec_Count_value_d) by bin(TimeGenerated, 1h)
    ```

* Outgoing bytes per second: (Replace `your_kafka_cluster_name` with your cluster name.)

    ```kusto
    metrics_kafka_CL 
    | where ClusterName_s == "your_kafka_cluster_name" and InstanceName_s == "kafka-BrokerTopicMetrics-BytesOutPerSec-Count" 
    | summarize AggregatedValue = avg(kafka_BrokerTopicMetrics_BytesOutPerSec_Count_value_d) by bin(TimeGenerated, 1h)
    ```

    You can also enter `*` to search all types logged. For a list of logs that are available for queries, see [Kafka workload](../monitor-hdinsight-reference.md#kafka-workload).

    :::image type="content" source="./media/apache-kafka-log-analytics-operations-management/apache-kafka-cpu-usage.png" alt-text="Apache kafka log analytics cpu usage." border="true":::

## Next steps

For more information on Azure Monitor, see [Azure Monitor overview](/azure/azure-monitor/overview), and [Query Azure Monitor logs to monitor HDInsight clusters](../hdinsight-hadoop-oms-log-analytics-use-queries.md).

For more information on working with Apache Kafka, see the following documents:

* [Mirror Apache Kafka between HDInsight clusters](apache-kafka-mirroring.md)
* [Increase the scale of Apache Kafka on HDInsight](apache-kafka-scalability.md)
* [Use Apache Spark streaming (DStreams) with Apache Kafka](../hdinsight-apache-spark-with-kafka.md)
* [Use Apache Spark structured streaming with Apache Kafka](../hdinsight-apache-kafka-spark-structured-streaming.md)
