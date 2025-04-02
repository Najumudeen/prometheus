# Prometheus Architecutre
------------------------

## Prometheus main components

1. `Retrieval` : Scrapes metric data
2. `TSDB` : Stroes metrics data
3. `HTTP Server` : Accepts PromQL Query

Exporters => Prometheus Targets

Pull Metrics

Pushgateway => Short lived Jobs

Service Discovery:

  - kubernetes
  * EC2

Discovery Targets

it's used to generate alerts. Not reponsile for set alerts

push alerts

it's sent alerts to alertmanager and alertmanager send notification to slack.

Collecting Metrics
------------------

Promethues collects metrics by sending http requests to /metrics endpoint of each target.

Prometheues can be configured to use a different path other then /metrics.

Prometheues send http_request to Targets / metrics


Exporters
----------

Most Systems by default don't collect metrics and expose them on an HTTP endpoint to be consumed by a prometheus server.

Exporter collect metrics and expose them in a format Prometheus expects.

exporter collect metrics from service, converts it into a format that prometheus expects and then exposes a /metrics endpoint
so that prometheus is able to scrap that data

Prometheus has several native exporters

|     Native Exporter            | 
|--------------------------------
|  Node Exporters(Linux servers) |
|  Windows                       |
|  Mysql                         | 
|  Apache                        | 
|  HAProxy                       |


Client Libraries

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


Pull Based Model
-----------------

Prometeus needs to have a list of all targets it should scrape.

Prometues   <>====>  target


Authentication & Encryption
---------------------------

Encypt the packets.

Node Exporter TLS
-----------------

Using open SSL to generate self-signed certificates or use encrpyt or veersion

[!COMMAND]
sudo openssl req -new -newkey rsa:2048 -days 365 -nodes -x509 -keyout node_exporter.key -out node_exporter.crt -subj \
"/C=US/ST=California/L=Oakland/O=MyOrg/CN=localhost" -addext \
"subjectAltName = DNS:localhost"

it will generated 2 files

node_exporter.key
node_exporter.crt

vi config.yml

tls_server_config:
  cert_file: node_exporter.crt
  key_file: node_exporter.key

# Update the node_exporter to use the cert file

./node_exporter --web.config=config.yml


msg="TLS enabled"

sudo mkdir /etc/node_exporter

mv node_exporter.* /etc/node_exporter

sudo cp config.yml /etc/node_exporter

chown -R node_exporter:node_exporter /etc/node_exporter

# Create node exporter service file

vi /etc/systemd/system/node_exporter.service

Use the file context: [node_exporter.service](node_exporter.service)

systemctl daemon-reload

systemctl restart node_exporter


curl https://localhost:9100/metrics

will get the error
Caused by self signed certificate

Then you have pass -k flags

curl -k https://localhost:9100/metrics

Error will gone

Prometheus TLS Config
---------------------

First thing, you have to copy node_exporter.crt Prometheus Server

# Use Scp to copy the file over

scp username:password@node:/etc/node_exporter/node_exporter.crt /etc/prometheus

# change the ownership of the file

chown prometheus:prometheus node_exporter.crt

vi /etc/prometheus/prometheus.yml

Then update the configuration

```
scrape_configs:
  - job_name: "node"
    scheme: https
    tls_config:
      ca_file: /etc/prometheus/node_exporter.crt
      insecure_skip_verify: true  # Only needed for self signed certs
    static_configs:
      - targets: [ "192.168.1.168:9100" ]

```

systemctl restart prometheus

Prometheus Authentication
-------------------------


# Generate Hash Password


Hashing
-------

# Instsll apache2-utils or httpd-tools

$ sudo apt install apache2-utils # install on your linux box

$ htpasswd -nBC 12 "" | tr -d ':\n'

# Use Preferred Programming Language

import getpass
import bcrypt

password = getpass.getpass("password: ")
hashed_password = bcrypt.hashpw(password.encode("utf-8"), bcrypt.gensalt())
print(hashed_password.decode())

Once you have a hash then go to Auth Configuration Node Exporter

vi /etc/node_exporter/config.yml

```
tls_server_config:
  cert_file: node_exporter.crt
  key_file: node_exporter.key
basic_auth_users:
  prometheus: #@#$$#$%#$%$%      <username:hashpassword>

```

systemctl restart node_exporter

login to prometheus server

will get the error because you have update only node exporter, now you have to update prometheus also.


vi /etc/prometheus/prometheus.yml

```
- job_name: "node"
  scheme: https
  basic_auth:
    username: prometheus
    password: password  <plantext password>

```
             
systemctl restart prometheus


Prometheus Metrics
-------

