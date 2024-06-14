# Logging with ELK Stack

## 目录

- [Introduction](#introduction)
- [Setting Up ELK Stack](#setting-up-elk-stack)
  - [Step 1: Clone the Repo](#step-1-clone-the-repo)
  - [Step 2: Spin Up the ELK Stack](#step-2-spin-up-the-elk-stack)
  - [Step 3: Change the Passwords](#step-3-change-the-passwords)
  - [Step 4: Check Docker Containers](#step-4-check-docker-containers)
- [Loading Test Data](#loading-test-data)
  - [UI Method](#ui-method)
  - [CLI Method](#cli-method)
- [Viewing the Data](#viewing-the-data)
- [Kibana Query Language (KQL)](#kql-knowledge)
- [Analyzing Data](#analyzing-data)
- [Integrating ELK with Filebeat](#integrating-elk-with-filebeat)
  - [Step 1: Fix Docker-ELK Repo](#step-1-fix-docker-elk-repo)
  - [Step 2: Rebuild and Run Docker-Compose](#step-2-rebuild-and-run-docker-compose)
  - [Step 3: Build Filebeat Docker Container](#step-3-build-filebeat-docker-container)
  - [Step 4: Run Filebeat Agent](#step-4-run-filebeat-agent)
  - [Step 5: Create Index Pattern in Kibana](#step-5-create-index-pattern-in-kibana)
- [Python Logging Levels](#python-logging-levels)
- [Hands-on: Logging in Python](#hands-on-logging-in-python)
- [Open Questions](#open-questions)
- [Logging Basics](#logging-basics)
- [What to Log](#what-to-log)
- [How to Log](#how-to-log)
- [Nginx Logs](#nginx-logs)
- [Fluentd](#fluentd)
- [A Taste of Splunk](#a-taste-of-splunk)

## Introduction

This guide provides a hands-on approach to setting up and using the ELK (Elasticsearch, Logstash, Kibana) stack for logging. It covers everything from cloning the repository to configuring and analyzing logs.

## Setting Up ELK Stack

### Step 1: Clone the Repo

First, clone the ELK Docker repository:

```bash
git clone https://github.com/deviantony/docker-elk
```

Quickly read through the repository and identify `logstash.conf`.

### Step 2: Spin Up the ELK Stack

Navigate to the `docker-elk` directory and start the ELK stack:

```bash
cd docker-elk
docker-compose up setup
docker-compose up
```

Expect to see something like this:

```
...
Creating docker-elk_elasticsearch_1 ... done
Creating docker-elk_kibana_1        ... done
Creating docker-elk_logstash_1      ... done
Attaching to docker-elk_elasticsearch_1, docker-elk_logstash_1, docker-elk_kibana_1
logstash_1       | OpenJDK 64-Bit Server VM warning: Option UseConcMarkSweepGC was deprecated in version 9.0 and will likely be removed in a future release.
elasticsearch_1  | Created elasticsearch keystore in /usr/share/elasticsearch/config/elasticsearch.keystore
kibana_1         | {"type":"log","@timestamp":"2020-07-20T11:03:35Z","tags":["warning","plugins-discovery"],"pid":7,"message":"Expect plugin \"id\" in camelCase, but found: apm_oss"}
```

The stack is pre-configured with the following privileged bootstrap user:

```text
user: elastic
password: changeme
```

### Step 3: Change the Passwords

Execute the following commands to change the passwords for the stack users:

```bash
docker-compose exec elasticsearch bin/elasticsearch-reset-password --batch --user elastic
docker-compose exec elasticsearch bin/elasticsearch-reset-password --batch --user logstash_internal
docker-compose exec elasticsearch bin/elasticsearch-reset-password --batch --user kibana_system
```

Sample output:

```text
❯ docker-compose exec elasticsearch bin/elasticsearch-reset-password --batch --user elastic
Password for the [elastic] user successfully reset.
New value: Wb7Yv6niUXEayOtNMCl*

❯ docker-compose exec elasticsearch bin/elasticsearch-reset-password --batch --user logstash_internal
Password for the [logstash_internal] user successfully reset.
New value: 3cHnndylWE0Dm3eHNx6N

❯ docker-compose exec elasticsearch bin/elasticsearch-reset-password --batch --user kibana_system
Password for the [kibana_system] user successfully reset.
New value: DzA=zWV8dxvTqsm=MVNS
```

Update the `.env` file in `docker-elk` with the new passwords:

```env
ELASTIC_VERSION=8.2.3

## Passwords for stack users
ELASTIC_PASSWORD='Wb7Yv6niUXEayOtNMCl*'
LOGSTASH_INTERNAL_PASSWORD='3cHnndylWE0Dm3eHNx6N'
KIBANA_SYSTEM_PASSWORD='DzA=zWV8dxvTqsm=MVNS'
```

Restart the service:

```bash
docker-compose down
docker-compose up
```

### Step 4: Check Docker Containers

Ensure the containers are running and check their logs:

```bash
docker ps
```

Example output:

```text
CONTAINER ID   IMAGE                      COMMAND                  CREATED         STATUS         PORTS                                                                                            NAMES
7054d9fafeb9   docker-elk_logstash        "/usr/local/bin/dock…"   7 minutes ago   Up 7 minutes   0.0.0.0:5000->5000/tcp, 0.0.0.0:5044->5044/tcp, 0.0.0.0:9600->9600/tcp, 0.0.0.0:5000->5000/udp   docker-elk_logstash_1
d506ff044997   docker-elk_kibana          "/bin/tini -- /usr/l…"   7 minutes ago   Up 7 minutes   0.0.0.0:5601->5601/tcp                                                                           docker-elk_kibana_1
2dba2aed0b46   docker-elk_elasticsearch   "/bin/tini -- /usr/l…"   7 minutes ago   Up 7 minutes   0.0.0.0:9200->9200/tcp, 0.0.0.0:9300->9300/tcp                                                   docker-elk_elasticsearch_1
```

The ELK stack consists of Filebeat (installed on your webapp servers), Logstash, Elasticsearch, and Kibana.

- **Filebeat**: Lightweight shipper for forwarding and centralizing log data.
- **Logstash**: Server-side data processing pipeline.
- **Elasticsearch**: Distributed search and analytics engine.
- **Kibana**: Frontend application for search and data visualization.

## Loading Test Data

### UI Method

1. Login to `localhost:5601`.
2. Go to Home (`http://localhost:5601/app/kibana#/home`).
3. Add Sample Data -> Sample web logs -> Add data.

### CLI Method

Send log entries via TCP:

```bash
# Using GNU netcat (CentOS, Fedora, MacOS Homebrew, ...)
$ cat /path/to/logfile.log | nc -c localhost 5000

# Using BSD netcat (Debian, Ubuntu, MacOS system, ...)
$ cat /path/to/logfile.log | nc -q0 localhost 5000
```

Create an index pattern via Kibana API:

```bash
$ curl -XPOST -D- 'http://localhost:5601/api/saved_objects/index-pattern' \
    -H 'Content-Type: application/json' \
    -H 'kbn-version: 7.8.0' \
    -u elastic:<your generated elastic password> \
    -d '{"attributes":{"title":"logstash-*","timeFieldName":"@timestamp"}}'
```

Or via the UI -> "Connect to your Elasticsearch index" -> Prefix "logstash-\*" and select `@timestamp`.

## Viewing the Data

Navigate to `http://localhost:5601/app/kibana#/discover`.

Explore the logs or logstash indexes.

## KQL Knowledge

Kibana Query Language (KQL) filters:

```text
request: /kibana
response: 5*
host: *elastic* and not response:200
referer: http\://facebook.com*
```

Escape special characters with `\`.

## Analyzing Data

1. Go to Dashboards: `http://localhost:5601/app/kibana#/dashboard`.
2. Create New -> Lens.

**Task**: Understand the health of the webapp by analyzing request statuses.

Questions:

- Where do most requests come from over the last 7 days? (Location and IP wise)
- What is the percentage of error logs?
- What are the top requests?
- What extension is used the most?
- What browser do most customers use?

## Integrating ELK with Filebeat

### Step 1: Fix Docker-ELK Repo

In the `docker-elk` folder, edit `logstash.conf` to add:

```text
index => "%{[@metadata][beat]}-%{[@metadata][version]}"
```

Example:

```text
output {
  elasticsearch {
    hosts => "elasticsearch:9200"
    user => "elastic"
    password => "changeme"
    index => "%{[@metadata][beat]}-%{[@metadata][version]}"
  }
}
```

### Step 2: Rebuild and Run Docker-Compose

```bash
docker-compose build
docker-compose up
```

Check the running containers:

```bash
docker-compose ps
```

### Step 3: Build Filebeat Docker Container

```bash
cd filebeat
docker build -t fcc/filebeat .
```

### Step 4: Run Filebeat Agent

```bash
docker run --rm -v '/var/lib/docker/containers:/usr/share/dockerlogs/data:ro' -v '/var/run/docker.sock:/var/run/docker.sock' --name filebeat fcc/filebeat:latest
```

### Step 5: Create Index Pattern in Kibana

1. Login to Kibana.
2. Go to `Stack Management` -> `Kibana` -> `Data View` -> `Create`.
3. Type `filebeat-*` and select `@timestamp`.
4. Click "Create data view".

## Homework

1. Research how to set up an alert for the logging system, e.g., alert when the number of 5xx > 10.
2. Investigate setting up dashboards with Terraform.

## Python Logging Levels

Six log levels in Python:

- NOTSET = 0
- DEBUG = 10
- INFO = 20
- WARN = 30
- ERROR = 40
- CRITICAL = 50

## Hands-on: Logging in Python

### Step 1: Create a Logger File

Create `my_logger.py`:

```python
import logging
import sys
from logging.handlers import TimedRotatingFileHandler

FORMATTER = logging.Formatter("%(asctime)s — %(name)s — %(levelname)s — %(message)s")
LOG_FILE = "my_app.log"

def get_console_handler():
    console_handler = logging.StreamHandler(sys.stdout)
    console_handler.setFormatter(FORMATTER)
    return console_handler

def get_file_handler():
    file_handler = TimedRotatingFileHandler(LOG_FILE, when='midnight')
    file_handler.setFormatter(FORMATTER)
    return file_handler

def get_logger(logger_name):
    logger = logging.getLogger(logger_name)
    logger.setLevel(logging.DEBUG)  # better to have too much log than not enough
    logger.addHandler(get_console_handler())
    logger.addHandler(get_file_handler())
    logger.propagate = False
    return logger
```

### Step 2: Introduce the Logger to Your Main Program

In `flask_app.py`, add:

```python
from my_logger import get_logger
log = get_logger(__name__)

@app.route('/test/')
def test():
    log.info('hitting /test/ endpoint')
    return 'rest'

@app.errorhandler(500)
def handle_500(error):
    log.error(f'something went wrong {error}')
    return str(error), 500
```

### Step 3: Try It Out

Run locally:

```bash
python flask_app.py
```

Or via Docker Compose.

### Step 4: Figure Out the Problems

Ensure the `my_app.log` file captures stack traces by wrapping functions with try-except.

### Step 5: Turn All Logs into One Liner

Modify `my_logger.py`:

```python
class OneLineExceptionFormatter(logging.Formatter):
    def formatException(self, exc_info):
        result = super().formatException(exc_info)
        return repr(result)

    def format(self, record):
        result = super().format(record)
        if record.exc_text:
            result = result.replace("\n", "")
        return result
```

Update `get_logger` to use `OneLineExceptionFormatter`:

```python
def get_logger(logger_name):
    logger = logging.getLogger(logger_name)
    logger.setLevel(logging.DEBUG)
    one_line_formatter = OneLineExceptionFormatter("%(asctime)s — %(name)s — %(levelname)s — %(message)s")
    logger.addHandler(get_console_handler(one_line_formatter))
    logger.addHandler(get_file_handler(one_line_formatter))
    logger.propagate = False
    return logger
```

Rebuild and rerun to see one-liner logs.

### Step 6: Send Logs to Kibana

Run your app or container and generate traffic to see logs in Kibana.

## Open Questions

- What is logging?
- What log types do we know?
- If you are a developer, what do you use the logs for?
- Which tool do you use for logging?
- How do you tell if your application is buggy or not by log?
- How frequently do you collect the logs?
- What if you have millions of logs, how do you find the useful one?

## Logging Basics

- Visibility is key for your application.

## What to Log

### Authentication, Authorization, and Access

Example logs:

- TIMESTAMP
- ACCESS_METHOD
- STATUS_CODE
- RESPONSE_TIME

### Changes

Example logs:

- OS image version update
- Network/interface update
- Kernel update

### Availability

Example logs:

- Healthcheck
- Startup/shutdown logs

### Resource

Example logs:

- Exhausted threads/connections
- Exhausted CPU/disk/memory log

### Threats

Example logs:

- Hacker input
- Common User Behavior

## How to Log

Modern logging systems:

1. Logging Aggregator/Forwarder
2. Search
3. Visualization

## Nginx Logs

Create logs folder and files:

```bash
sudo mkdir /usr/share/nginx/logs
sudo touch /usr/share/nginx/logs/error.log
sudo touch /usr/share/nginx/logs/access.log
```

Restart nginx:

```bash
sudo systemctl reload nginx
```

Check logs:

```bash
less /usr/share/nginx/logs/error.log
less /usr/share/nginx/logs/access.log
```

## Fluentd

Install Fluentd:

```bash
sudo apt-get install ruby-full
gem install fluentd
fluentd -s conf
fluentd -c conf/fluent.conf &
```

Install Fluentd UI:

```bash
gem install fluentd-ui
fluentd-ui setup
fluentd-ui start --daemonize
```

Install multiline parser and configure Fluentd for Nginx logs:

```bash
fluent-gem install fluent-plugin-multi-format-parser
cp fluent.conf /conf/fluent.conf
sudo pkill -f fluentd
fluentd -c conf/fluent.conf -vv &
```

# A Taste of Splunk

## Step1. Download

Fill in the following details and download

## Step2. Install

If there is a GUI, click install; otherwise, run

```
sudo dpkg -i splunk_xxx.deb
```

## Step3. Run splunk

At the first time running splunk, it will ask for the initial setup for the admin account.

```
cd /opt/splunk/bin/
sudo ./splunk start --accept-license
```

## Step4. Login to splunk

Use the username and password that you set in the last step to login

## Step5. Download the data

Let us download some sample data
https://data.montgomerycountymd.gov/
https://data.montgomerycountymd.gov/Finance-Tax-Property/County-Spending/vpf9-6irq

## Step6. Add the data to splunk

Add Data -> upload from my computer -> select \*.csv from ~/Download

## Step7. A Few Selection

**Source type**: this helps Splunk to learn what data you have got.
_ You specify a directory as a data source.
_ You specify a network input as a data source. \* You specify a data source that has been forwarded from another Splunk instance.

Let us select "Automatic" in this short example.

**Host**: When the Splunk platform indexes data, each event receives a "host" value. The host value should be the name of the machine from which the event originates. The type of input you choose determines the available configuration options.

**Index**:The Splunk platform stores incoming data as events in the selected index. Consider using a "sandbox" index as a destination if you have problems determining a source type for your data. A sandbox index lets you troubleshoot your configuration without impacting production indexes. You can always change this setting later.

Let us use the default.

Now choose review

## Step9. You should see an error

The error is about Splunk cannot recognise the time format
go to `/opt/splunk/etc/system/local/props.conf`

add the following lines

```
TIMESTAMP_FIELDS = Invoice Date
TIME_FORMAT=%m/%d/%Y
```

and restart Splunk

```
cd /opt/splunk/bin/
./splunk restart
```

### Step10. Reload the data

Now there should be no error. Let us submit the data and start searching!
