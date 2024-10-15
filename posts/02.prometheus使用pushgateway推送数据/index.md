# Prometheus使用Pushgateway推送数据

## Pushgateway简介

Prometheus 的 Pushgateway 是一个简单的 HTTP 服务器，它允许数据被推送到该服务器，而不是通过拉取的方式获取。它的存在是为了让临时和批处理作业能够将其指标暴露给 Prometheus。由于这类作业可能存在的时长不足以被主动抓取，因此它们可以将指标推送到 Pushgateway。随后，Pushgateway 会将这些指标暴露给 Prometheus。

Pushgateway 作为中间件，保存推送的数据直到 Prometheus 抓取。它支持从多个来源推送指标，**每个来源都通过唯一的 `job` 标签来标识，并且可以选择性地附加额外的标签**。

Pushgateway GitHub 地址：[https://github.com/prometheus/pushgateway](https://github.com/prometheus/pushgateway)

## 安装

要安装 Pushgateway，你可以下载二进制包或使用包管理器，但更推荐使用 Docker。你可以在任何机器上安装 Pushgateway，通常只需要一台 Pushgateway 服务器即可处理来自所有来源的指标。以下是使用 Docker 设置 Pushgateway 的方法：

```shell
docker pull prom/pushgateway

docker run -d -p 9091:9091 prom/pushgateway
```

## 向 Pushgateway 推送指标

向 Pushgateway 推送指标时，你可以使用 `curl` 命令行工具或者开发自定义应用程序发送 HTTP 请求。此外，还有适用于多种编程语言的第三方库，可简化向 Pushgateway 发送指标的过程。

### 使用 `curl`

以下是一个向 Pushgateway 推送单个指标的例子：

```shell
curl -X POST http://{pushgateway_server}:{port}/metrics/job/myjob/instance/myinstance \
     --data &#39;my_metric{label=&#34;value&#34;} 1.0&#39;
```

此命令推送了一个名为 `my_metric` 的指标，其值为 `1.0` 并带有一个 `label` 设置为 `value` 的标签。

### 使用第三方库

有若干第三方库可以帮助你将 Pushgateway 的功能整合到你的应用程序中。这些库提供了一个更高层次的 API 来发送指标，使得与 Pushgateway 的交互更加容易管理。

例如，在 Python 中，你可以使用 `prometheus_client` 库，下面是一段实现代码：
```python
import csv
from prometheus_client import CollectorRegistry, Gauge, push_to_gateway

class PrometheusPusher:
    def __init__(self, metric_name: str, description: str, job_name: str, pushgateway_url: str = &#39;localhost:9091&#39;):
        &#34;&#34;&#34;
        Initialize an instance of PrometheusPusher.
        
        :param metric_name: The name of the metric.
        :param description: A description of the metric.
        :param job_name: Job name used to identify the source.
        :param pushgateway_url: URL of the Pushgateway service, default is localhost:9091.
        &#34;&#34;&#34;
        self.metric_name = metric_name
        self.description = description
        self.job_name = job_name
        self.pushgateway_url = pushgateway_url
        self.registry = CollectorRegistry()
        self.gauge = None

    def create_gauge(self, label_names: list):
        &#34;&#34;&#34;
        Create a gauge metric with labels.
        
        :param label_names: List of label names.
        &#34;&#34;&#34;
        self.gauge = Gauge(self.metric_name, self.description, label_names, registry=self.registry)
    
    def push_metrics(self, label_values: list):
        &#34;&#34;&#34;
        Push the metric value to the Pushgateway.
        
        :param label_values: List of label values.
        &#34;&#34;&#34;
        if not self.gauge:
            print(&#39;Error: Gauge is not created&#39;)
            return
        self.gauge.labels(*label_values).set(1)
        try:
            push_to_gateway(self.pushgateway_url, job=self.job_name, registry=self.registry)
            print(f&#39;Successfully pushed metrics for {label_values}&#39;)
        except Exception as e:
            print(f&#39;Failed to push metrics for {label_values}. Error: {e}&#39;)
    
    def push_metrics_from_csv(self, csv_file_path: str):
        &#34;&#34;&#34;
        Read data from a CSV file and push metrics.
        
        :param csv_file_path: Path to the CSV file.
        &#34;&#34;&#34;
        with open(csv_file_path, mode=&#39;r&#39;) as file:
            reader = csv.reader(file)
            # Get the label names (first row)
            label_names = next(reader)
            self.create_gauge(label_names)
            for row in reader:
                if len(row) != len(label_names):
                    print(f&#34;Warning: Ignoring row with incorrect number of columns: {row}&#34;)
                    continue
                self.push_metrics(row)

# Example CSV file format:
# label1, label2
# value1, value2
# ...

# Main entry point
if __name__ == &#39;__main__&#39;:
    # Set CSV file path and other parameters
    csv_file_path = &#39;example_data.csv&#39;
    metric_name = &#39;example_metric&#39;
    description = &#39;An example metric for demonstration purposes.&#39;
    job_name = &#34;example_job&#34;
    pushgateway_url = &#39;slcx-grafana.calix.local:9091&#39;

    # Create an instance of PrometheusPusher and push data from CSV file
    pusher = PrometheusPusher(metric_name, description, job_name, pushgateway_url)
    pusher.push_metrics_from_csv(csv_file_path)
```

---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://hezephyr.github.io/posts/02.prometheus%E4%BD%BF%E7%94%A8pushgateway%E6%8E%A8%E9%80%81%E6%95%B0%E6%8D%AE/  

