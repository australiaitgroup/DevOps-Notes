# Monitoring in DevOps - II

## 目录

- [Linux System Stats Breakdown](#linux-system-stats-breakdown)
  - [vmstat](#vmstat)
  - [uptime](#uptime)
  - [top (htop)](#top-htop)
- [Managing Grafana Dashboard with Terraform](#managing-grafana-dashboard-with-terraform)
  - [Goal](#goal)
  - [Steps](#steps)
- [Setup a Traffic Generator](#setup-a-traffic-generator)
  - [Goal](#goal-1)
  - [Steps](#steps-1)
- [Integrate Grafana with OpsGenie](#integrate-grafana-with-opsgenie)
- [Build a Comprehensive Dashboards and Alerting System](#build-a-comprehensive-dashboards-and-alerting-system)

## Linux System Stats Breakdown

### vmstat

`vmstat` reports describe the current state of a Linux system. Information regarding the running state of a system is useful when diagnosing performance-related issues.

```
vmstat [interval] [count]
```

```
vagrant@linux:/home/vagrant$ vmstat 1 20
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0   1804 296112 132244 1207392    0    0    21   119   53  130  0  0 99  0  0
 0  0   1804 296104 132244 1207392    0    0     0     0   98  217  0  0 100  0  0
 0  0   1804 296104 132244 1207392    0    0     0     0  106  243  0  1 99  0  0
 0  0   1804 296104 132244 1207392    0    0     0     0   70  220  0  0 100  0  0
 0  0   1804 296104 132244 1207392    0    0     0     0   66  215  0  0 100  0  0
 0  0   1804 296104 132244 1207392    0    0     0     0   66  221  0  1 100  0  0
 0  0   1804 296104 132244 1207392    0    0     0     0   62  230  0  0 100  0  0
 0  0   1804 296104 132244 1207392    0    0     0     0   73  235  0  0 100  0  0
 0  0   1804 296104 132244 1207392    0    0     0     0   65  244  0  0 100  0  0
 0  0   1804 296104 132244 1207392    0    0     0     0   73  241  0  0 100  0  0
 0  0   1804 296104 132244 1207392    0    0     0     0   80  243  0  0 100  0  0
 0  0   1804 296104 132244 1207392    0    0     0     0  109  231  0  0 99  0  0
 0  0   1804 296104 132244 1207392    0    0     0     0   84  225  0  0 100  0  0
 0  0   1804 296104 132244 1207392    0    0     0     0   94  285  1  0 99  0  0
 0  0   1804 296104 132244 1207392    0    0     0     0   66  234  0  0 100  0  0
 0  0   1804 296104 132244 1207392    0    0     0     0   76  257  0  0 100  0  0
 0  0   1804 296104 132244 1207392    0    0     0     0   65  237  0  0 100  0  0
 0  0   1804 296104 132244 1207392    0    0     0     0   65  247  0  0 100  0  0
 0  0   1804 296104 132244 1207392    0    0     0     0   75  250  0  0 100  0  0
 0  0   1804 296104 132244 1207392    0    0     0     0  114  327  0  0 100  0  0
```

More readable: Convert the number to Megabytes (e.g. 1M=1000KB=1000000B).

```
vagrant@linux:/home/vagrant$ vmstat -S m 1 10
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b      swpd  free   buff  cache  si  so  bi  bo  in  cs us sy id wa st
 1  0        1   303    135   1236   0   0   21 118  53 130  0  0 99  0  0
 0  0        1   303    135   1236   0   0    0   0 103 248  0  1 100 0  0
 0  0        1   303    135   1236   0   0    0

 0 97 239  0  0 100 0  0
 0  0        1   303    135   1236   0   0    0    0 91 249  0  0 100 0  0
 0  0        1   303    135   1236   0   0    0    0 106 272 0  0 100 0  0
 0  0        1   303    135   1236   0   0    0    0 113 252 0  0 100 0  0
 0  0        1   303    135   1236   0   0    0    0 91 233  0  0 100 0  0
 0  0        1   303    135   1236   0   0    0    0 92 246 0  0 100 0  0
 0  0        1   303    135   1236   0   0    0    0 86 234 0  0 100 0  0
 0  0        1   303    135   1236   0   0    0    0 96 266 0  0 100 0  0
```

For 1M=1024KB, use `-S M` instead.

### uptime

```
vagrant@linux:/home/vagrant$ uptime
15:22:38 up  5:26,  2 users,  load average: 0.00, 0.04, 0.06
```

`uptime` gives a one-line display of the following information: the current time, how long the system has been running, how many users are currently logged on, and the system load averages for the past 1, 5, and 15 minutes.

Load > # of CPUs may mean CPU saturation.

### top (htop)

System and per-process interval summary:

```
Tasks: 142 total,   1 running, 106 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.0 us,  0.2 sy,  0.0 ni, 99.8 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  2041120 total,   295024 free,   406084 used,  1340012 buff/cache
KiB Swap:  2097148 total,  2095344 free,     1804 used.  1453912 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
17229 root      20   0  907880  44200  25148 S   0.7  2.2   0:18.58 containerd
19715 vagrant   20   0   41804   3696   3072 R   0.3  0.2   0:00.01 top
    1 root      20   0  159956   9080   6504 S   0.0  0.4   0:04.49 systemd
    2 root      20   0       0      0      0 S   0.0  0.0   0:00.00 kthreadd
    4 root       0 -20       0      0      0 I   0.0  0.0   0:00.00 kworker/0:0H
    6 root       0 -20       0      0      0 I   0.0  0.0   0:00.00 mm_percpu_wq
    7 root      20   0       0      0      0 S   0.0  0.0   0:00.28 ksoftirqd/0
    8 root      20   0       0      0      0 I   0.0  0.0   0:00.58 rcu_sched
    ...
```

- %CPU is summed across all CPUs
- Can miss short-lived processes (atop won’t)
- Can consume noticeable CPU to read /proc

## Managing Grafana Dashboard with Terraform

### Goal

- Use Grafana to manage a team's dashboards.

### Steps

1. **Spin up your Grafana server locally**

```
cd flask_statsd_prometheus
docker build -t jr/flask_app_statsd .
docker-compose -f docker-compose.yml -f docker-compose-infra.yml up
```

If you have trouble spinning it up, follow the last week's lecture notes to clean up your environment first.

2. **Prepare a `main.tf` and `variables.tf` file**

```
cd ..
mkdir terraform_grafana
touch main.tf
touch variables.tf
```

Your folder structure should now look like this:

```
WK8_Monitoring_2
|_docs
|_...
|_terraform_grafana
    |_main.tf
    |_variables.tf
```

3. **Find the provider**

Go to [Terraform Registry](https://registry.terraform.io/) and search for Grafana.
Click on the "grafana/grafana verified".

4. **Copy the content to `main.tf`**

5. **Fill in the options to connect to Grafana**

Check out [the example usage](https://registry.terraform.io/providers/grafana/grafana/latest/docs) and fill in the `main.tf` as follows:

```hcl
terraform {
  required_providers {
    grafana = {
      source  = "grafana/grafana"
      version = "1.14.0"
    }
  }
}

provider "grafana" {
  url  = "http://localhost:3000"
  auth = var.grafana_auth
}
```

Fill in the `variables.tf` as follows:

```hcl
variable "grafana_auth" {
  type    = string
  default = "admin:foobar"
}
```

This is to simplify the demonstration. You should avoid exposing your username/password.

6. **Format and validate**

```bash
terraform init
terraform fmt
terraform validate
```

You should see:

```
Success! The configuration is valid.
```

7. **Check what you can do with the Grafana provider**

Visit [Terraform Grafana Provider Documentation](https://registry.terraform.io/providers/grafana/grafana/latest/docs/resources/dashboard).

8. **Share your already created dashboard as JSON**

Open `localhost:3000` -> Manage -> Dashboards -> Your recently created dashboard -> Share dashboard

Export -> Save to file

Move it under `terraform_grafana/`

```
mv ~/Downloads/<your dashboard.json> .
```

9. **Add the dashboard to your Terraform config**

In `main.tf`, add the following:

```hcl
resource "grafana_dashboard" "metrics" {
  overwrite   = true
  config_json = file("Your dashboard.json")
}
```

10. **Try apply**

```bash
terraform fmt
terraform validate
terraform apply
```

Try to `terraform destroy` the current one and redo the `terraform apply`.

Now, no matter what others may change in your dashboard, you can simply recover it by `terraform apply`.

## Setup a Traffic Generator

### Goal

- Generate traffic automatically to avoid manual page refresh.
- Have an automated way to trigger alerts.

### Steps

1. **Write a quick `traffic_generator.py`**

Identify the two endpoints that you would like to test: `/test/` and `/test1/`.

```
mkdir traffic_generator
cd traffic_generator
touch traffic_generator.py
```

Add the following content to `traffic_generator.py`:

```python
from locust import HttpUser, task, between

class QuickstartUser(HttpUser):
    wait_time = between(5, 9)  # Simulated users wait between 5 and 9 seconds

    @task
    def index_page(self):
        self.client.get("/test1/")

    @task(5)
    def view_organisation(self):
        self.client.get("/test/")

    def on_start(self):
        pass
```

Reference: [Locust Writing a Locustfile](https://docs.locust.io/en/stable/writing-a-locustfile.html)

2. **Run the traffic generator**

If you have Python and Locust installed, you can simply run:

```bash
locust -f traffic_generator.py
```

Open [http://0.0.0.0:8089/](http://0.0.0.0:8089/) and fill in the load test with the following parameters:

The last entry is [http://localhost:5000](http://localhost:5000).

Assuming

you have started the `flask_statsd_prometheus`, click "Start swarming".

3. **Adjust the parameters and observe the differences**

- What if you change the weight of the task?
- What if you change the number of users?
- What if you change the spawn rate?
- What can you observe from Grafana?

## Integrate Grafana with OpsGenie

1. **Register an OpsGenie Free Account**

Register a free 14-day trial account by filling in the required details and giving a name for the site.

2. **Wait for the new instance to spin up**

Once it is done, you should see the following page:

3. **Configure your profile**

Put your phone number in and send test notifications. Do you receive a phone call?

4. **Set up a team**

Give your team a name and invite yourself to be the owner.

5. **Enable integration**

Select Grafana for the integration.

Once saved, click integration.

You should now see the URL and API key.

6. **Spin up Grafana**

Follow these steps to spin up the web app, Prometheus, and Grafana:

```bash
cd flask_statsd_prometheus
docker-compose -f docker-compose.yml -f docker-compose-infra.yml up
```

Go to `localhost:3000` and navigate to Alerting -> Notification Channels.

Copy-paste the API key and API URL from step 5 and give it a name: `JiangRenMainAlert` (or any name you prefer).

Save and test.

7. **Create a dashboard and a chart**

Create a new dashboard and name it WebApp.

Add a panel and name it WebApp Error Rate.

Ensure it is selected as Prometheus for the data source.

To calculate the error rate, use the following formula:

```hcl
sum (rate(request_count{status="500"}[15s])) / sum (rate(request_count[15s]))
```

You should have generated enough data on `localhost:5000/test` and `localhost:5000/test1` from the previous steps.

8. **Configure alert for the chart**

Evaluate this alert every 15 seconds for 1 minute and set a condition that if the value is above 0.5 (50%) for 10s, then alert.

Save and hit `localhost:5000/test1`.

Do you receive an alert on your phone?

I received a phone call, an email, and a message in the OpsGenie portal.

If you have the OpsGenie App, it can also push notifications.

9. **Other configurations**

Currently, the alert is a default alert. Of course, you could customize it for you and your team. To adjust the notification order or time, go to settings -> notification and configure the change alerts here.

For a team schedule, go to Teams -> <Your Team> -> On-call. Try to invite a classmate and schedule an on-call for him/her.

For any change made for the team, you can track the logs here:

## Questions

Are you able to make P1 and P2 alerts call you immediately?
Hint: Search for `grafana opsgenie priority` on Google.

## Build a Comprehensive Dashboards and Alerting System

### Goal

Let us start with the WebApp that we have.

One thing we must be clear:

- Monitor -> For analysis
- Alerts -> For issues

Alert is just a subset of Monitor.

Each alert should map to a response action.

### Starting from the 4 Golden Signals

According to the 4 golden signals, we need Error Rate, Throughput, Latency, Saturation.

These 4 golden signals can be used as alerts.

- Would the monitor show a good picture of the WebApp when we only introduce the error rate?
- What about the healthy response?
- How do I tell if every host/region/url is functional?

- Should we set alerts on them if they are not functional? What problems do we see?
- How do we set a threshold? How do we make the alert less noisy?

### Build the Monitors First

You need to gain enough visibility to decide which metrics with which labels can be used for alerts.

Let us build an all-inclusive Throughput, Success/Error Rate, Error Count, and Latency dashboard, which includes all the labels for now.

### Filter and Play Around with the Threshold

The labels are your good companions; use them to filter your data. You may need to try different math for the threshold calculation.

For example, avg() is going to slow down the trigger of an alert. There are min(), max(), and many other functions that could help with triggering alerts faster.

### Test the Alerts

Make sure you have tested the alerts yourself before making them available to prod. You can:

- Set up a testbed environment.
- Trigger the alerts via a script.
- Channel the alerts to Slack to avoid being paged.
- Review the alerts.

### When an Alert is Triggered, What Do We Do?

An alert means that the system may not be able to recover itself and requires human intervention.

## Questions

- Are you able to create a dashboard with variables that can monitor a WebApp comprehensively?
  - Hint: [Grafana Variables](https://grafana.com/docs/grafana/latest/variables/)
- Are you able to monitor container saturation (CPU, memory, disk of the WebApp container)?
  - Hint: [Prometheus cAdvisor](https://prometheus.io/docs/guides/cadvisor/)
