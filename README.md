# Prometheus Architecutre



![image](https://github.com/user-attachments/assets/aeefcb83-698a-4ed8-be3d-df6a9e86b1e7)



## Table Of Content

- [Prometheus main components](#prometheus-main-components)
    - [Prometheus Targets](#prometheus-targets)
    - [Pull Metrics](#pull-metrics)
    - [Pushgateway](#pushgateway)
    - [Service Discovery](#service-discovery)
    - [Discovery Targets](#discovery-targets)
    - [Collecting Metrics](#collecting-metrics)
    - [Exporters](#exporters)
    - [Client Libraries](#client-libraries)
    - [Pull Based Model](#pull-based-model)
- [Authentication and Encryption](#authentication-and-encryption)
- [Create node exporter service file](#create-node-exporter-service-file)
- [Use scp to copy the file over](#use-scp-to-copy-the-file-over)
- [Generate Hash Password](#generate-hash-password)
- [Hashing](#hashing)
- [Use Preferred Programming Language](#use-preferred-programming-language)
- [Prometheus Metrics](#prometheus-metrics)
- [Metric Attributes](#metric-attributes)
    - [Metrics Type](#metrics-type)
    - [Metrics Rules](#metrics-rules)
- [How Setup Prometheus on Docker container](#how-setup-prometheus-on-docker-container)
- [What is Promtools](#what-is-promtools)
- [PromQL](#promql)
    - [PromQL Data Types](#promql-data-types)
    - [Selectors](#selectors)
    - [Matchers](#matchers)
- [Aggregation](#aggregation)
      

## Prometheus main components

1. `Retrieval` : Scrapes metric data
2. `TSDB` : Stroes metrics data
3. `HTTP Server` : Accepts PromQL Query

### Prometheus Targets

### Pull Metrics

### Pushgateway 

Short lived Jobs

### Service Discovery

  - kubernetes
  * EC2

### Discovery Targets

it's used to generate alerts. Not reponsile for set alerts

push alerts

it's sent alerts to alertmanager and alertmanager send notification to slack.

### Collecting Metrics

Promethues collects metrics by sending http requests to /metrics endpoint of each target.
Prometheues can be configured to use a different path other then /metrics.
Prometheues send http_request to Targets / metrics


### Exporters

Most Systems by default don't collect metrics and expose them on an HTTP endpoint to be consumed by a prometheus server.
Exporter collect metrics and expose them in a format Prometheus expects.

Exporter collect metrics from service, converts it into a format that prometheus expects and then exposes a /metrics endpoint
so that prometheus is able to scrap that data

Prometheus has several native exporters

|     Native Exporter            | 
|--------------------------------
|  Node Exporters(Linux servers) |
|  Windows                       |
|  Mysql                         | 
|  Apache                        | 
|  HAProxy                       |


### Client Libraries

Can we monitror application metrics

1. Number of errors/exceptions
2. Latency of requests
3. Job execution time

Prometheus comes with client libraries that allow you to expose any application metrics you need prometheus to track.

|     Language support       | 
|-----------------------------
|      Go                    |
|      Java                  |
|      Pyhton                |
|      Ruby                  |
|      Rust                  |


### Pull Based Model

Prometeus needs to have a list of all targets it should scrape.

Prometues  <====>  target

## Authentication and Encryption

Encypt the packets.

Node Exporter TLS

Using open SSL to generate self-signed certificates or use encrpyt or veersion

```
sudo openssl req -new -newkey rsa:2048 -days 365 -nodes -x509 -keyout node_exporter.key -out node_exporter.crt -subj "/C=US/ST=California/L=Oakland/O=MyOrg/CN=localhost" -addext "subjectAltName = DNS:localhost"
```

it will generated 2 files

```
node_exporter.key
node_exporter.crt
```

```
vi config.yml

tls_server_config:
  cert_file: node_exporter.crt
  key_file: node_exporter.key
```

Update the node_exporter to use the cert file

./node_exporter --web.config=config.yml


msg="TLS enabled"

```
sudo mkdir /etc/node_exporter
mv node_exporter.* /etc/node_exporter
sudo cp config.yml /etc/node_exporter
```

```
chown -R node_exporter:node_exporter /etc/node_exporter
```

## Create node exporter service file

vi /etc/systemd/system/node_exporter.service

Use the file context: [node_exporter.service](node_exporter.service)

```
systemctl daemon-reload
systemctl restart node_exporter
```

curl https://localhost:9100/metrics

will get the error
Caused by self signed certificate

Then you have pass -k flags

curl -k https://localhost:9100/metrics

Error will gone

Prometheus TLS Config

First thing, you have to copy node_exporter.crt Prometheus Server

## Use scp to copy the file over

scp username:password@node:/etc/node_exporter/node_exporter.crt /etc/prometheus

change the ownership of the file

chown prometheus:prometheus node_exporter.crt

Then update the configuration

```
vi /etc/prometheus/prometheus.yml
scrape_configs:
  - job_name: "node"
    scheme: https
    tls_config:
      ca_file: /etc/prometheus/node_exporter.crt
      insecure_skip_verify: true  # Only needed for self signed certs
    static_configs:
      - targets: [ "192.168.1.168:9100" ]
```

```
systemctl restart prometheus
```

Prometheus Authentication

## Generate Hash Password

## Hashing

Install apache2-utils or httpd-tools

```
sudo apt install apache2-utils # install on your linux box
```

```
htpasswd -nBC 12 "" | tr -d ':\n'
```

## Use Preferred Programming Language

```Python

import getpass
import bcrypt

password = getpass.getpass("password: ")
hashed_password = bcrypt.hashpw(password.encode("utf-8"), bcrypt.gensalt())
print(hashed_password.decode())
```

Once you have a hash then go to Auth Configuration Node Exporter

```
vi /etc/node_exporter/config.yml

tls_server_config:
  cert_file: node_exporter.crt
  key_file: node_exporter.key
basic_auth_users:
  prometheus: #@#$$#$%#$%$%  <username:hashpassword>

```

```
systemctl restart node_exporter
```

Login to prometheus server

will get the error because you have update only node exporter, now you have to update prometheus also.

```
vi /etc/prometheus/prometheus.yml

- job_name: "node"
  scheme: https
  basic_auth:
    username: prometheus
    password: password  <plantext password>

```

```
systemctl restart prometheus
```

## Prometheus Metrics

<metric_name>[{<label_1="value_1>,<label_N="value_N">}]<metric_value>

  node_cpu_seconds_total{cpu="0",mode="idle"} 258277.86

 Labels (cpu 0,1,2,3) provide us information on which cpu this metric is for what `cpu state(idle)`.
 
```
   node_cpu_seconds_total{cpu="0",mode="idle"} 258244.86
   node_cpu_seconds_total{cpu="1",mode="idle"} 428277.86
   node_cpu_seconds_total{cpu="2",mode="idle"} 288277.86
   node_cpu_seconds_total{cpu="3",mode="idle"} 258202.86
```

### Timestamp

when Prometheus scraps a target and retrieves metrics, it also stores the time at which the metric was scraped as well.

The time stamp will look like this: `1668215300`

This is called a unix timestamp, which is the number of seconds that have elapsed since `Epoch(January 1st 1970 UTC)`.

### Prometheus Time Series

Stream of timestamped values sharing the same metric and set of labels.

Any Metric with a unique set of labels, as we collect data for that over time that's going to be just called a `time series`.

```
Example:
   node_filesystem_files{device="sda2", instance="server1}  ................. series1
   node_filesystem_files{device="sda3", instance="server1}  ................. series2
   node_filesystem_files{device="sda2", instance="server2}  ................. series3
   node_filesystem_files{device="sda3", instance="server2}  ................. series4
                                                          Scrap Interval
   node_cpu_seconds_total{cpu="0", instance="server1"}  ................. series5
   node_cpu_seconds_total{cpu="1", instance="server1"}  ................. series6
   node_cpu_seconds_total{cpu="0", instance="server2"}  ................. series7
   node_cpu_seconds_total{cpu="1", instance="server2"}  ................. series8
```

There are two metrics(node_filesystem_files, node_cpu_seconds_total)
There are 8 total time series(unique combination of metrics and set of labels)

## Metric Attributes

Metrics have a TYPE and HELP attributes

> [!NOTE]
> HELP - description of what the metrics is TYPE - Specifies what type of metric(counter, gauge, histogram, summary)


### Metrics Type

1. `Counter`
2. `Gauge`
3. `Histogram`
4. `Summary`


#### <ins>Counter</ins>

How many times did X happen<br/>
Number can only increase<br/>
For this a counter metric would be used as it would count the number of seconds a process has been running for. The uptime of a process can never go down, so a gauge metric shouldn’t be used.<br/>

Good for

Total # Requests<br/>
Total # Exceptions<br/>
Total # of job executions<br/>

#### <ins>Gauge</ins>

What is the current value of X<br/>
Can go up or down depending on the current metrics value is

Current CPU Utilization<br/>
Available System Memory<br/>
Number of concurrent requests<br/>

#### <ins>Histogram</ins>

Histograms should be used to calculate how long or how big something is and allows you to group observations into configurable bucket sizes.<br/>

You want to track?<br/>

Respone Time (< 1s < 0.5s < 0.2s ) How many total request completed within less than 0.5 seconds.<br/>
Request Size (< 1500Mb, < 1000Mb, < 800Mb) that fell under 1,000 megabytes, what was the total number of requests that fell under 1,500 megabytes?<br/>

#### <ins>Summary</ins>

Similar to histograms(track how long or how big)
How many observations fell below x
Don't have to define quantiles ahead of time

Response Time(20% = .3s, 50% = 0.8s, 80% = 1s) what summary's going to do is it's going to give us percentages?<br/>
Request Size(20% = 50Mb, 50% = 200Mb, 80% = 500Mb)


### Metrics Rules

Metric name specifies a general feature of a system to be measured.<br/>
May contain ASCII letters, numbers, underscores, and colons.<br/>
Must match the regex [a-zA-Z_:][a-zA-Z0-9_:]*<br/>
Colons are reserved only for recording rules<br/>

#### <ins>Labels</ins>

labels are Key-Value pairs associated with a metric.<br/>
Allows you to split up a metric by a specified criteria<br/>
Metric can have more than one label<br/>
ASCII leters, numbers, underscores<br/>
Must match regex[a-zA-Z0-9_]*<br/>

Example: cpu=0, cpu=1, cpu=2, cpu=3, fs=/data. fs=/root, fs=/dev, fs=/run

Why would we want to use labels?

Difficult to calculate total requests across all paths

Example:

/auth  <==> requests_auth_total<br/>
/user <==> requests_user_total<br/>
/products <==> requests_products_total<br/>
/cart <==> requests_cart_total<br/>
/orders <==> request_orders_total<br/>

Instead you have to calculate Sum all requests: `sum(requests_total)` using the below using path<br/>

requests_total{path=/auth}<br/>
requests_total{path=/user}<br/>
requests_total{path=/products}<br/>
requests_total{path=/cart}<br/>
requests_total{path=/orders}<br/>


### <ins>Multiple Labels</ins>

paricular endpoint with multiple HTTP methods calls

requests_total{path=/auth,method=get}<br/>
requests_total{path=/auth,method=post}<br/>
requests_total{path=/auth,method=patch}<br/>
requests_total{path=/auth,method=delete}<br/>

### <ins>Internal Labels</ins>

Metric name is just another label

```node_cpu_seconds_total{cpu=0} = {__name__=node_cpu_seconds_total,cpu=0}```

label surrounded by two underscore are considered internal to prometheus.<br/>

### Labels

Every metric is assigned 2 labels by default(instance and job)<br/>
node_boot_time_seconds{instance="192.168.1.168:9100",job="node"}<br/>
Here instance is represent targets and job is job_name in the config.yaml file.<br/>
Each unique combination of metrics & labels is a separate time series.<br/>

## How Setup Prometheus on Docker container

```
docker run -d /path-to/prometheus-docker.yml:/etc/prometheus/prometheus.yml -p 9090:9090 prom/prometheus
```

Default port Prometheus listen on 9090.

## What is Promtools

Promtools is a utility tool shipped with Prometheus that can be used to:

Check and validate configuration<br/>
  Validate Prometheus.yml<br/>
  validate rules files<br/>

Validate metric passed to it are correctley formatted.
Can Perform queries on a Prometheus server.
Debugging & Profiling a Prometheus server.
Perform unit tests against Recording/Alerting rules.

```
promtool check config /etc/prometheus/prometheus.yml

Checking prometheus.yml
 SUCCESS: prometheus.yml is valid prometheus config file syntax

```

Prometheus configs can now be validated before applying them to a production server.

This will prevent any unnecessary downtime while config issues are being identified.

As we progress through this course, we will cover all of the other features promtools has to offer.


Monitoing Container with `cadvisor` 


## PromQL

What is PromQL?

Short for Prometheus QUery Language
Main way to query metrics within Prometheus
Data returned can be visualized in dashboards.
Used to build alerting rules to notify administrators.

### Section Outline

Expression Data Structure<br/>
Selectors & modifiers<br/>
Operators & Functions<br/>
Vector Matching<br/>
Aggregators<br/>
Subqueries<br/>
Histogram/Summary<br/>


### PromQL Data Types

A PromQL expression can evaluate to one of four types:

1. String - a simple string value (currentely unused)
2. Scalar - a simple numeric floating point value
3. Instant Vector - set of time series containg a single sample for each time series, all sharing the same timestamp
4. Range Vector - set of time series containing a range of data points over time for each time series.

#### <ins>String</ins>

some random text<br/>
This is a string<br/>

#### <ins>Scalar</ins>

54.743<br/>
127.43<br/>

#### <ins>Instant Vector</ins>

Query: node_cpu_seconds_total<br/>

specific metric with all of its unique labels. Every combination of metric and unique labels is going to be one time series.

Each time series return value. it's going to return one `single point` sample for each time series, and they are all going to be at the same exact timestamp.

#### <ins>Range Vector</ins>

Query: node_cpu_seconds_total[3m]<br/>

You will get more than `one TimeStamp Values`.

### Selectors

A query with just the metric name will return all time series with that metric.


### Matchers

What if we only want to return a subset of time series for a metric.

```
Label Matchers

= Equality Matcher - Exact match on Label value
!= Negative Equality Matcher - return time series that don't have the label.
=~ Regular Expression <atcher - matches time series with labels that match regex.
!~ Negative regular expression matcher
```

### <ins>Equality Matcher</ins>

Return all time series from node01

```
node_filesystem_avail_bytes{instance="node01}
```
> [!NOTE]
> instance="node01" will match all time series from node01.


### <ins>Negative Equality Matcher</ins>

Return all time series where device is not equal to `tmpfs`.

```
node_filesystem_avail_bytes{device!="tmpfs"}
```
> [!NOTE]
> != matcher will ensure that only device that ate not `tmpfs`.

### <ins>Regular Expression matcher</ins>

Return all time series where device starts with `/dev/sda` (sda2 & sda3)

```
node_filesystem_avail_bytes{device=~"/dev/sda.*"}
```
> [!NOTE]
> To Match anything that starts with /dev/sda, a regex dev/sda.* will need to be used.

> [!TIP]
> [REGEX TIPS](https://github.com/google/re2/wiki/syntax)

### <ins>Negative Equality Matcher</ins>

Return all time series where mount point does not start with `/boot`.

```
node_filesystem_avail_bytes{mountpoint!~/boot.*"}
```

> [!NOTE]
> To match anything that starts with "/boot", a regex "/boot.*" will need to be used.

### Multiple Selectors

Return all time series from node1 without a `device=tmpfs`

```
node_filesystem_avail_bytes{instance="node01",device!="tmpfs"}
```

> [!NOTE]
> Multiple selectors can be used by separating them by a comma

### Range Vector Selectors

Returns all the values for a metrics over a period of time.

```
node_arp_entries{instance="node1"}[2m]
```

>[!NOTE]
> Returns node_arp_entries metric data for the past 2 minutes

### Offset Modifiers 

when performing a query, it returns the current value of a metric

```
node_memory_Active_bytes{instance="node01"}
```

To get historic data use an `offset modifier` after the label matching

You can go and find back in time?

```
node_memory_Active_bytes{instance="node01"}offset 5m
```
22259302 Value 5 minutes ago

### Time units


|    Suffix        |    Meaning                    |
|   :-----         |   :-----                      |
| ms               |    Milliseconds               |
| s                |    Seconds                    |
| m                |    Minutes                    |
| h                |    Hours                      |
| d                |    Days                       |
| w                |    Weeks                      |
| y                |    Years, which have 365 days |


####  5 days ago

```
node_memory_Active_bytes{instance="node01"}offset 5d
```

#### 2 weeks ago
```
node_memory_Active_bytes{instance="node01"}offset 2w
```
#### 1.5 hours ago
```
node_memory_Active_bytes{instance="node01"}offset 1h30m
```

#### To go back to a specific point in time use the `@modifier`
```
node_memory_Active_bytes{instance="node01"}@1663265188  Unix timestamp
```

### The offset modifier can be combined with the `@modifier`

```
node_memory_Active_bytes{instance="node01"}@1663265188 offset 5m     # 5 minutes before
```

### Order does not matter when combining `@modifier` and `offset modifier`

```
node_memory_Active_bytes{instance="node01"} offset 5m @1663265188
```

### The offset and @modifier also work with range vectors
```
node_memory_Active_bytes{instance="node01"}[2m] @1663265188 offset 10m
```

> [!TIP]
> [EpochConverter](https://www.epochconverter.com/)

Which of the following queries will return the 1h ago available memory bytes on node01:9100 host?

```
node_memory_MemAvailable_bytes{instance="node01:9100"} offset 1h
```
### Arithmetic Operators

Arithmetic operators provide the ability to perform basic math operations.

|    Operator  |    Description   |
|   :-----     |   :-----         |
|    +         |   Addition       |
|    -         |   Subtraction    |
|    *         |   Multiplication |
|    /         |   Division       |
|    %         |   Modulo         |
|    ^         |   Power          |


> [!TIP]
> Now PromQL supports several different operators.

The `+` operator will add x amount to the result

```
node_memory_Active_bytes{instance="node1"} + 10         Add 10 to result
```

> [!TIP]
> By default the amount of active bytes return value in `bytes`. Convert bytes to kilobytes node_memory_Active_bytes / 1024.

### <ins>Comparison Operators</ins>

|    Operator  |    Description   |
|   :-----     |   :-----         |
|   ==         |  Equal           |
|   !==        |  Not Equal       |
|   >          |  Greater Than    |
|   <          |  Less than       |
|   >=         |  Greater or Equal|
|   <=         | Less or equak    |

<ins>Filter result for anything grater than 100</ins>

```
node_network_flags > 100
```

<ins>Filter all interfaces that have received 220 or greater netwrok packets.</ins>

```
node_network_receive_packets_total >= 220
```

<ins>Bool Operator can be used to return a `true(1)` or `false(0)` result. To find which filesystems have less
than 1000 bytes available:</ins>

```
node_filesystem_avail_bytes < bool 1000   result 1 or 0
```

> [!TIP]
> Bool Operators are mostly used to generating alerts


### Binary operator precedence

When an PromQL expression has multiple binary operators, they follow an `order of precedence`, from highest to lowest

1. `^`
2. `*, /, %, atan2`
3. `+, -`
4. `==, !=, <=, <, >=, >`
5. `and, unless`
6. `or`

Operators on the same precedence level are `left-associative`.

For Example, 2 * 3 % 2 is equivalent to ( 2 * 3) % 2

However `^` is `right associative`, so 2 ^ 3 ^ 2 is equivalent to 2 ^ (3 ^ 2)

### Logical Operators

PromQL has 3 logical operators

1. `OR`
2. `AND`
3. `UNLESS`

### <ins>AND Operator</ins>

Return all time series greater than 1000 and less than 3000

```
node_filesystem_avail_bytes > 1000 `and` node_filesystem_avail_bytes < 3000
```

### <ins>OR Operator</ins>

Return all time series less than 500 or greater than 70000

```
node_filesystem_avail_bytes < 500 `or` node_filesystem_avail_bytes > 70000
```

### <ins>UNLESS Operator</ins>

Unless operator results in a vector consisting of elements on the left side for which there are no elements on the right side.

Return all vectors greater than 1000 unless they are greater than 30000

```
node_filesystem_avail_bytes > 1000 `unless` node_filesystem_avail_bytes > 30000
```

### Vector Matching Keywords

Operators between and `instant vectors` and `scalars`.

```
node_filesystem_avail_bytes < 1000      instant vectors is `node_filesystem_avail_bytes` and `1000` is scalars
``` 

Operatos between and `2 instant vectros`

```
node_filesystem_avail_bytes / node_filesystem_size_bytes * 100   return percentage that's free 50
```

different types of vector matching

1. `One-To-One`
2. `One-To-Many/Many-To-One`

Samples with exactly the same labels get matched together

```
node_filesystem_avail_bytes{instance="node01",job="node",mountpoint="/home"}
node_filesystem_size_byte{instance="node01",job="node",mountpoint="/home"}
```
> [!TIP]
> All labels must be same for samples to match

There may be certain instances where an operation needs to be performed on 2 vectors with `differing labels`.

```
http_erros
http_erros{method="get", code="500"}      40

http_requests
http_requests{method="get"}               421

2 labels: {method, code}          1 label: {method}
http_errors{code="500"}      /    http_requests
```
> [!WARNING]
> If we try to perform this query we're not going to get any results. `NO MATCH`

<ins>Solution</ins>

`Ignoring keyword` can br used to "ignore" an labels to ensure there is a match between 2 vectors.

```
http_erros
http_erros{method="get", code="500"}      40

http_requests
http_requests{method="get"}               421

http_errors{code="500"}      /    ignoring(code) http_requests
{method="get"} 0.0950              // 40 / 421

```

Ignoring keyword is used to ignore a label when matching, the `on` keyword is to specify exact list of labels to match on

```
http_erros
http_erros{method="get", code="500"}      40

http_requests
http_requests{method="get"}               421

http_errors{code="500"}      /    ignoring(code) http_requests
                          or
http_errors{code="500"}      /    on(method) http_requests

List of all labels to match on
```
             
`One-To-One vector matching` - Every element in the vector on the left of the operator tries to find a single matching element on the right

```

{cpu=0,mode=idle} 2           {cpu=0,mode=idle} 4          {cpu=0,mode=idle} 6
{cpu=0,mode=user} 5           {cpu=0,mode=user} 6          {cpu=0,mode=user} 11
{cpu=0,mode=user} 1      +    {cpu=0,mode=user} 3      =   {cpu=0,mode=user} 4
{cpu=0,mode=user} 7           {cpu=0,mode=user} 3          {cpu=0,mode=user} 10
  
       Vector1                      Vector2

```

`Many-To-One vector matching` - each vector elements on the one side can match with multiple elements on the many side

```

#### group_left

     Many                                  One

{error=400, path=/cats} 2                                                 {error=400, path=/cats} 4
{error=500, path=/cats} 5                                                 {error=500, path=/cats} 7 
{error=400, path=/dogs} 1        +        {path=/cats} 2          =       {error=400, path=/dogs} 8
{error=500, path=/dogs} 7                 {path=/dogs} 7                   {error=500, path=/dogs} 14

     http_errors              +    on(path)  `group_left` http_requests

```

### group_right

Group_right is the opposite of group_left, tells PromQL that elements from the left side are now matched with multiple elements from the right.

```

 One                                  Many
                                  {error=400, path=/cats} 2            {error=400, path=/cats} 4
 {path=/cats} 2                   {error=500, path=/cats} 5            {error=500, path=/cats} 7 
 {path=/dogs} 7           +       {error=400, path=/dogs} 1      =     {error=400, path=/dogs} 8
                                  {error=500, path=/dogs} 7            {error=500, path=/dogs} 14

  http_requests           +     on(path)  group_right http_reqeusts

  ```

  ## Aggregation

  Aggregation operators, allow you to take an instant vector and aggregate its elements, resulting in a new instant vector, with fewer elements.

|  Aggregator  |   Description                                               |
|   :-----     |   :-----                                                    |
|  Sum         |   Calculate sum over dimensions                             |
|  Min         |   Select minimum over dimensions                            |
|  Max         |   Select maximum over dimensions                            |
|  Avg         |   Average over dimensions                                   |
|  Group       |   All values in the resulting vector are 1                  |
|  Stddev      |   Calculate population standard deviations over dimensions  |
|  Stdvar      |   Calculate population standard variance over dimensions    |
|  Count       |   Count number of elements in the vector                    |
|  Count_values|   Count number of elements with same value                  |
|  Bottomk     |   Smallest k elements by sample value                       |
|  Topk        |   Largest k elements by sample value                        |
|  Quantile    |   calculate -quantile (0 <= p <= 1) over dimensions         |

```

http_requests

sum(http_requests)

max(http_requests)

avg(http_requests)

```

### by clause

The `by` clause allows you to choose which labels to aggregate along

```
$ http_requests
http_requests(method="get", pah="/auth") 3
http_requests(method="post", pah="/auth") 1

$ sum by(path) (http_requests)
{path="/auth"} 4  // 3 + 1
```

```
$ http_requests
http_requests(method="get", pah="/auth") 3
http_requests(method="get", pah="/auth") 1

$ sum by(method) (http_requests)
{method="get"} 4  // 3 + 1
```

Get the request per node

```
sum by(instance) (http_requests)
```

### without

The `without` keyword does the opposite of `by` and tells the query which labels not to include in the aggregation.

```
$ http_requests
http_requests{method="get", path="/auth", instance="node1"}    3
http_requests{method="post", path="/auth", instance="node1"}   1

$ sum without(path) (http_requests)
{instance="node01", method="get"} 3
{instance="node01", method="post"} 1
```

Aggregate on every label except `path`. The equivalent to `by(instance, method)`

```
sum by(instance, cpu) (node_cpu_seconds_total)
sum without(cpu, mode) (node_cpu_seconds_total)
```
